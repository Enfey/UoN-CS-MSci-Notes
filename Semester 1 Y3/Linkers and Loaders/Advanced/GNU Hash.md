Consulted Sources:
https://sourceware.org/git/?p=binutils-gdb.git;a=blob;f=gold/dynobj.h;h=ee08c201354e9f128cf272ffebbe4930f1f60ccb;hb=HEAD#l94
https://gnu.googlesource.com/binutils-gdb/%2B/refs/heads/gdb-15-branch/bfd/elflink.c#6437
https://stackoverflow.com/questions/14467173/bit-setting-and-bit-shifting-in-ansi-c
# Purpose
A section with the name `.gnu.hash` of type `SHT_GNU_HASH` contains a hash table augmented with a  [bloom filter](https://en.wikipedia.org/wiki/Bloom_filter), auxilary data, and provides `.dynsym` ordering whereby ordering is via hash-values, such that memory access tends to be adjacent and monotonically increasing to exploit memory locality for improved cache behaviour. This is used to accelerate dynamic symbol lookups by the dynamic linker. The below structure permits a dynamic linker to avoid expensive string comparisons for most misses and find corresponding definition quickly for hits. 
## Layout
Conceptually, the the on-disk layout of `.gnu.hash` is as follows:
1. **Header**
   ```C
	uint32_t    nbuckets; 
	uint32_t    symndx;  // first sym participating in hashing
	uint32_t    maskwords; // number of words in bloom filter
	uint32_t    shift2; // secondary shift used by bf
	uint32_t    values[dynsymcount - symndx];
	```
2. **Bloom Filter**
	`uintX_t bloom_filter[maskwords];`
	Contains `maskwords` of machine-word sized entries; type is interpreted according to `ELFCLASS` bits. The bloom filter is indexed and two bit positions per hash are checked as a reject step.
3. **Bucket Array**
	`uint32_t buckets[nbuckets];`
	Contains `nbuckets` entries, heuristic chosen by linker, each containing first symbol index in that bucket's chain portion (or `STN_UNDEF (0)`).
4. **Chain Table**
	`uint32_t chain[values]`
	Variable length chain table. Flat array containing symbol indexes. One computes the location to begin searching `chain` sequentially via `chain_index = bucket_value - sym_index`.
## Instantiation
1. **Choose `symndx`**
	Symbols before this index are typically the null symbol, versioning metadata and so and so forth. For all symbols `i >= symndx`, the linker computes their hash and reorders them such that symbols with lower GNU hash values appear earlier (though no actual reordering of `.dynsym` occurs). The GNU hash algorithm is detailed below:
```C
	unsigned long
	bfd_eld_gnu_hash (const name *namearg){
		const unsigned char *name = (const unsigned char *)
		namearg;
		unsigned long h = 5381;
		unsigned char ch;
		
		while ((ch = *name++)!= '\0')
			h = (h << 5) + h + ch;
		return h & 0xffffffff;
	}
	```
2. **Compute size parameters**
	The static linker chooses (pseudo-`C`):
```C
		nbuckets  = prime(n_symbols) / 4; // Heuristic
		maskwords = power2((n_symbols - symndx) /
		ELFCLASS_BITS);
		shift2 = small constant; // Almost always 6
```
3. **Bloom Filter**
	For each `sym i`, such that `i>=symndx`:
```C
	uint32_t h         = gnu_hash(sym_name);
	uintX_t word_index = (h / ELFCLASS_BITS) & (maskwords 
	-1);
	uintX_t bit 1      = h % ELFCLASS_BITS;
	uintX_t bit 2      = (h >> shift2) % ELFCLASS_BITS;
	
	bloom[word_index]  |= (1UL << bit1) | (1UL << bit2);
```
4. **Fill buckets**
	For each `sym i`, such that `i>=symndx`:
```C
	uint32_t h = gnu_hash(sym_name);
	uint32_t b = h % nbuckets;

	if (buckets [b] == STN_UNDEF)
		buckets[b] = i; // first symbol with this bucket
```
5. **Fill chain table**
	Flat array parallel to the re-ordered `.dynsym`, containing one entry per symbol with index `i >= symndx`. Low bit encodes end of chain. We often store some auxilary info throughout our mechanical construction that aids our population of the chain table. These are all detailed below.
	```C
	count[nbuckets];  // number of symbols hashed to bucket b
	offset[nbuckets]; // starting index in chain for bucket b
	cursor[nbuckets]; // insertion pointer for bucket b
	
	// 1. Count num symbols per bucket
	// For each name_i >= symndx
	b = gnu_hash(name_i) % nbuckets;
	count[b]++;
	
	// 2. Compute where each bucket's chain entries will
	// appear in final chain table
	o = 0
	for (b = 0; b < nbuckets; b++){
		offset[b] = o;
		o += count[b];
	}
	
	// 3. Init cursor per bucket 
	cursor[b] = offset[b];
	
	// 4. Fill chain in linear pass
	h = gnu_hash(name_i);
	b = h % nbuckets;
	chain[cursor[b]] = h & ~1u; // hash with low bit cleared
	cursor[b]++;
	
	// 5. Mark end of bucket chains
	last = cursor[b] - 1
	chain[last] |= 1u   // set low bit = end of chain
	```
## Usage
1. **Compute GNU hash for requested name**
	`uint32_t h = gnu_hash(name);`
2. **Read .gnu.hash header parameters**
	Load first four words of the .gnu.hash section
	```C
	uint32_t nbuckets; 
	uint32_t symndx;      
	uint32_t maskwords;   // number of bloom filter words
	uint32_t shift2;      // secondary bloom filter shift 
	// (usually 6)
	```
	Also locate the bloom filter, bucket table, and chain table, as described in [layout](#layout).
3. **Set membership test**
	The bloom filter eliminates most misses without having to touch symbol table memory. 
	```C
	uintX_t word_index = (h / ELFCLASS_BITS) & (maskwords 
	-1);
	uintX_t bit 1      = h % ELFCLASS_BITS;
	uintX_t bit 2      = (h >> shift2) % ELFCLASS_BITS;
	
	// Load bloom word
	uintX_t bloom_word = bloom[word_index];
	
	if ((bloom_word & (1UL << bit1)) == 0 ||
    (bloom_word & (1UL << bit2)) == 0)
	{
	    // Impossible for this name to exist in this object
	    return NOT_FOUND;
	}
	```
3. **Bucket Lookup**
```C
	bucket = h % nbuckets;
	b = buckets[bucket];

	if (b == STN_UNDEF) 
		return NOT_FOUND;
	
	uint32_t chain_index = offset[b];

	for (;; chain_index++)
	{
		uint32_t c = chain[chain_index];
	
		// Compare the stored hash (low bit masked off)
	    if ((c & ~1u) == (h & ~1u)) {
	        // Hashes match, now compare symbol names
	        if (strcmp(name, dynstr[ dynsym[b].st_name ]) 
	        == 0)         
	        {
            // Typically additional checks occur:
            // versioning, binding/visibility, etc.
            return &dynsym[b];
	        }
	    }
	    // If low bit set, this is last symbol in chain
		if (c & 1u)
			break;
		b++; // Don't really understand this!
	}
```





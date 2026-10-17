### Master Theorem
1. $T(n) = 4T(n/2) +n$ 
	$a$ = $4$ 
	$b = 2$ 
	$f(n) = n$ 
	$log_2(4) = 2$ 
	$n^c = n^2$ 
	$f(n) \leq n^2$, thus case $1$, and $O(n^2)$ 
2. $T(n) = 4T(n/2) + n^2$ 
	2nd case
3. ...
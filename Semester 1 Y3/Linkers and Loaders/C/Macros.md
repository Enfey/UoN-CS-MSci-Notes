Textual substitution run before the compiler proper. 
Preprocessor handles `#include, #define, #if/#elif/#else/#endif, #ifdef, #ifndef, #undef, #pragma`
Macro expansion is purely textual, define in header and include wherever needed; define and include are the text substitution directives, conditional directives are the remaining. 

```c
//File = macros.h

#define BUFSIZE 4096

#define SQUARE(x) ((x)*(x))



```
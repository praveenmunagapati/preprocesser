# preprocesser :a c/c++ preprocessor library
this is c/c++ preprocess library. implements some arithmetic ,logical, code generation and loop basic macro function.
for example:
```c++
    A3D_PP_ADD(14,15); //just like 14+15 == 29
    A3D_PP_DIV(15,3);  //15 / 3 == 5
    A3D_PP_LESS(1,2);  //1 < 2 == 1
    #define INDEX(a)  cout<< a <<" ";
    A3D_PP_RANGE_CALL(1,10,1,INDEX); //will generate "1 2 3 4 5 6 7 8 9 10"
    ...
```
why we need do arithmetic operation like A3D_PP_ADD(14,15) instead of directly use 14+15?
14+15 will not get it's result before compile-time. 
```c++
        // macro expand period
        // A3D_PP_STRINGIZE( something ) == "something" 
        A3D_PP_STRINGIZE(14+15);            // will get "14+15"
        A3D_PP_STRINGIZE(A3D_PP_ADD(14,15)) // will get "29"
```
some times we don't have the true value of one number like in another macro and want to evaluate the add operation:
```c++
    #define DEFINE_CLASS(name, n) DEFINE_CLASS_HELPER(name,n)
    #define DEFINE_CLASS_HELPER(name, n) class name##n{};
    #define EXPAND_MULTI_CLASS(name, n)                \
                    DEFINE_CLASS(name, n)              \
    				DEFINE_CLASS(name, A3D_PP_ADD1(n)) \
    				DEFINE_CLASS(name, A3D_PP_ADD(n,2))
    EXPAND_MULTI_CLASS(A, 0) //we get class A0,A1,A2
    EXPAND_MULTI_CLASS(B, 1) //we get class B1,B2,B3
```
if we use n+1 instead of A3D_PP_ADD1(n) or n+2 instead of A3D_PP_ADD(n,2) will cause a compile error.
1. arithmetic operation
```c++
    //test macro + - * / %
	static_assert(A3D_PP_ADD1(2) == 3, "");
	static_assert(A3D_PP_ADD1(255) == 255, "");
	static_assert(A3D_PP_ADD(34, 100) == 134, "");
	static_assert(A3D_PP_ADD(255, 4) == 255, "");
	static_assert(A3D_PP_SUB1(0) == 0, "");
	static_assert(A3D_PP_SUB1(1) == 0, "");
	static_assert(A3D_PP_SUB(255, 18) == 237, "");
	static_assert(A3D_PP_SUB(17, 18) == 0, "");
	static_assert(A3D_PP_MUL(1, 5) ==  5, "");
	static_assert(A3D_PP_MUL(13, 4) == 52, "");
	static_assert(A3D_PP_MUL(15, 5) == 75, "");
	static_assert(A3D_PP_MUL(15, 17) == 255, ""); //mul   out of range
	static_assert(A3D_PP_MUL(16, 17) == 255, ""); //shift out of range
	static_assert(A3D_PP_DIV(15, 5) == 3,"");
	static_assert(A3D_PP_DIV(23, 4) == 5, "");
	static_assert(A3D_PP_DIV(23, 24) == 0, "");
	static_assert(A3D_PP_MOD(15, 5) == 0, "");
	static_assert(A3D_PP_MOD(23, 4) == 3, "");
	static_assert(A3D_PP_MOD(23, 24) == 23, "");
```
2. logical operation
```c++
    //test macro ==,!=,>,>=,<,<=
	static_assert(!A3D_PP_EQUAL(255, 16), "");
	static_assert( A3D_PP_EQUAL(16, 16), "");
	static_assert( A3D_PP_NEQUAL(17, 16), "");
	static_assert(!A3D_PP_NEQUAL(16, 16), "");
	static_assert( A3D_PP_GREATER(17, 16), "");
	static_assert(!A3D_PP_GREATER(16, 16), "");
	static_assert(!A3D_PP_GREATER(15, 16), "");
	static_assert( A3D_PP_GEQUAL(17, 16), "");
	static_assert( A3D_PP_GEQUAL(16, 16), "");
	static_assert(!A3D_PP_GEQUAL(15, 16), "");
	static_assert(!A3D_PP_LESS(17, 16), "");
	static_assert(!A3D_PP_LESS(16, 16), "");
	static_assert( A3D_PP_LESS(15, 16), "");
	static_assert(!A3D_PP_LEQUAL(17, 16), "");
	static_assert( A3D_PP_LEQUAL(16, 16), "");
	static_assert( A3D_PP_LEQUAL(15, 16), "");
```
3. enumerate operation, sequence is data-struct like (1,2,3,4,...),enumerate operation call a macro function on each element of sequence.
```c++
    #define  OUTPUT(a, b) cout << "  "<< a << A3D_PP_STRINGIZE(b) <<endl << endl; 
    #define FOREACH_CALL(it,...) |it|
    OUTPUT(" seq foreach call: \n", A3D_PP_FOREACH_ITEM(FOREACH_CALL, (int, char, short, float)));  //will output "|float| |short| |char| |int|"
    OUTPUT(" seq foreach tuple: \n ", A3D_PP_FOREACH_TUPLE(TEST_TUPLE, ((1, 2), (3, 4), (A3D_PP_NULL, p, 3), (0, A3D_PP_NULL, 4), (1, 1, 4), (2, 3, 4)), -)); //will output "|1,2,-| |3,4,-| |A3D_PP_NULL,p,3, -| |0,A3D_PP_NULL,4, -| |1,1,4, -| |2,3,4, -|"
```
4. enumeration helper function, help to compose complex sequence
```c++
    OUTPUT("1. seq expand: \n", A3D_PP_EXPAND(A3D_PP_EXPAND((1, 2, 3), 4),5));
	OUTPUT("2. seq unpack: \n", A3D_PP_UNPACK((1, 2, 3)));
	OUTPUT("3. seq compose: \n", A3D_PP_COMPOSE((1, 2), (3, 4)));
	OUTPUT("4. seq compose size: \n", A3D_PP_SIZE(A3D_PP_COMPOSE((1, 2), (3, 4))));
	OUTPUT("5. seq compose at: \n", A3D_PP_AT(A3D_PP_COMPOSE((1, 2), (3, 4)), 1));
	OUTPUT("6. seq compose 3: \n", A3D_PP_COMPOSE3((1, 2), (3, 4), (5,6) ));
	OUTPUT("7. seq compose with ex-item: \n", A3D_PP_COMPOSE_EX((unsigned, signed), (char, int), void ) );
	OUTPUT("8. seq compose3 with ex-item: \n", A3D_PP_COMPOSE3_EX((unsigned, signed), (char, int),(&,&&), void));
```
5. code generate
```c++
    //generate code like typename T0, typename T1, typename T2,...
    OUTPUT("12. for range prefix: \n", A3D_PP_RANGE_PREFIX(typename T, 2, 0, (,)));
    //generate code like typename T0, typename T2, typename T4...
	OUTPUT("13. for range prefix with step: \n", A3D_PP_RANGE_PREFIX_STEP(typename T, 0, 4, 2, (,)));
	//generate code like typename T0=ingore_t, typename T1=ingore_t...
	OUTPUT("14. for range wrap: \n", A3D_PP_RANGE_WRAP(typename T, 2, 0, = ingore_t, (,)));
	//generate code like typedef A0::type B0; typedef A1::type B1...
	OUTPUT("15. for range alias: \n", A3D_PP_RANGE_ALIAS(typedef A, ::type, B, 0, 2, (;)));
	//generate code like typedef A0::type B1; typedef A1::type B2..
	OUTPUT("16. for range chain alias: \n", A3D_PP_RANGE_CHAIN_ALIAS(typedef A, A3D_PP_NULL, B, 1, 3, (;)));
```
    more infomation please ref to test_preprocessor.cpp
    
    
	
    
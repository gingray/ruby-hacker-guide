# VALUE

I want to tell you about VALUE the most basic ruby conception if you take a look on sources it will be present everywhere. Example of C representation of method `Enumerable#select`:

```c
static VALUE
enum_find_all(VALUE obj)
{
    VALUE ary;

    RETURN_SIZED_ENUMERATOR(obj, 0, 0, enum_size);

    ary = rb_ary_new();
    rb_block_call(obj, id_each, 0, 0, find_all_i, ary);

    return ary;
}
```

What is about as you can see a we accept magic `VALUE obj` on input and return `VALUE ary` on output. Actually `VALUE` is 32-bit or 64-bit int depend on architecture.
Most of the time it acts as pointer but not all of the time. Let me show how actual structure looks like.

`VALUE` represent basic objects like `true`,`false`,`nil` they treated as constant here example

```c
//ruby.h

enum ruby_special_consts {
#if USE_FLONUM
    RUBY_Qfalse = 0x00,		/* ...0000 0000 */
    RUBY_Qtrue  = 0x14,		/* ...0001 0100 */
    RUBY_Qnil   = 0x08,		/* ...0000 1000 */
    RUBY_Qundef = 0x34,		/* ...0011 0100 */

    RUBY_IMMEDIATE_MASK = 0x07,
    RUBY_FIXNUM_FLAG    = 0x01,	/* ...xxxx xxx1 */
    RUBY_FLONUM_MASK    = 0x03,
    RUBY_FLONUM_FLAG    = 0x02,	/* ...xxxx xx10 */
    RUBY_SYMBOL_FLAG    = 0x0c,	/* ...0000 1100 */
#else
    RUBY_Qfalse = 0,		/* ...0000 0000 */
    RUBY_Qtrue  = 2,		/* ...0000 0010 */
    RUBY_Qnil   = 4,		/* ...0000 0100 */
    RUBY_Qundef = 6,		/* ...0000 0110 */

    RUBY_IMMEDIATE_MASK = 0x03,
    RUBY_FIXNUM_FLAG    = 0x01,	/* ...xxxx xxx1 */
    RUBY_FLONUM_MASK    = 0x00,	/* any values ANDed with FLONUM_MASK cannot be FLONUM_FLAG */
    RUBY_FLONUM_FLAG    = 0x02,
    RUBY_SYMBOL_FLAG    = 0x0e,	/* ...0000 1110 */
#endif
    RUBY_SPECIAL_SHIFT  = 8
};
```

You can ask what is `USE_FLONUM` its optimization for working with floating numbers for 64-bit systems introduced long time ago in short for storing float types in `VALUE` for performance improvements before it was stored as an regular ruby object [more information](https://bugs.ruby-lang.org/issues/6763)
Ok but how ruby detect is `VALUE` is just a plain int or it's a pointer? Actually by first 3-bits if it's 64-bit architecture or 2-bits if its 32-bit architecture. Here the code that 
checks which type actually `VALUE` is

```c
//ruby.h

static inline int
rb_type(VALUE obj)
{
    if (RB_IMMEDIATE_P(obj)) {
	if (RB_FIXNUM_P(obj)) return RUBY_T_FIXNUM;
        if (RB_FLONUM_P(obj)) return RUBY_T_FLOAT;
        if (obj == RUBY_Qtrue)  return RUBY_T_TRUE;
	if (RB_STATIC_SYM_P(obj)) return RUBY_T_SYMBOL;
	if (obj == RUBY_Qundef) return RUBY_T_UNDEF;
    }
    else if (!RB_TEST(obj)) {
	if (obj == RUBY_Qnil)   return RUBY_T_NIL;
	if (obj == RUBY_Qfalse) return RUBY_T_FALSE;
    }
    return RB_BUILTIN_TYPE(obj);
}
```

After check that `VALUE` is not special value ruby try to check which type of object `VALUE` is 
```c
//ruby.h
enum ruby_value_type {
    RUBY_T_NONE   = 0x00,

    RUBY_T_OBJECT = 0x01,
    RUBY_T_CLASS  = 0x02,
    RUBY_T_MODULE = 0x03,
    RUBY_T_FLOAT  = 0x04,
    RUBY_T_STRING = 0x05,
    RUBY_T_REGEXP = 0x06,
    RUBY_T_ARRAY  = 0x07,
    RUBY_T_HASH   = 0x08,
    RUBY_T_STRUCT = 0x09,
    RUBY_T_BIGNUM = 0x0a,
    RUBY_T_FILE   = 0x0b,
    RUBY_T_DATA   = 0x0c,
    RUBY_T_MATCH  = 0x0d,
    RUBY_T_COMPLEX  = 0x0e,
    RUBY_T_RATIONAL = 0x0f,

    RUBY_T_NIL    = 0x11,
    RUBY_T_TRUE   = 0x12,
    RUBY_T_FALSE  = 0x13,
    RUBY_T_SYMBOL = 0x14,
    RUBY_T_FIXNUM = 0x15,
    RUBY_T_UNDEF  = 0x16,

    RUBY_T_IMEMO  = 0x1a, /*!< @see imemo_type */
    RUBY_T_NODE   = 0x1b,
    RUBY_T_ICLASS = 0x1c,
    RUBY_T_ZOMBIE = 0x1d,
    RUBY_T_MOVED  = 0x1e,

    RUBY_T_MASK   = 0x1f
};
```
Which function/macros are available to work with `VALUE` short list of them
```c
void rb_check_type(VALUE,int); //check value have proper type if not raise exception
int rb_type(VALUE obj); // return obj type
rb_type_p(obj, type); return true if obj match type
char *rb_string_value_ptr(volatile VALUE*); // return pointer to a char sequence from string VALUE
VALUE rb_str_new(const char *ptr, long len); // create ruby string from char*
VALUE rb_sprintf(const char *format, ...); // create ruby string from formated *char
VALUE rb_data_typed_object_wrap(VALUE klass, void *datap, const rb_data_type_t *type); // function wrap user structure to a ruby object
```



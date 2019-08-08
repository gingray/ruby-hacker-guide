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

What is about as you can see a we accept magic `VALUE obj` on input and return `VALUE ary` on output. Actually `VALUE` is 32-bit or 64-bit int depend on architecture. Most of the time it acts as pointer but not all of the time. Let me show how actual structure looks like.

`VALUE` represent basic objects like `true`,`false`,`nil` they treated as constant here example

{% code-tabs %}
{% code-tabs-item title="ruby.h" %}
```c
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
{% endcode-tabs-item %}
{% endcode-tabs %}

You can ask what is `USE_FLONUM` its optimization for working with floating numbers for 64-bit systems introduced long time ago in short for storing float types in `VALUE` for performance improvements before it was stored as an regular ruby object [more information](https://bugs.ruby-lang.org/issues/6763)


# Unity3D优化_字符串拼接0GC

<!--more-->

{{< admonition tip "Unity3D优化_字符串拼接0GC" false >}}
{{< /admonition >}}
# Unity3D优化_字符串拼接0GC


## 字符串池缓存机制
- 为了避免大型项目中出现巨量的重复字符串，C#底层使用缓存池机制，池中每一个不同内容的字符串存在一个实例。

- 入池的字符串程序运行期间无法清理。所以并非所有的字符串都自动入池，也并非所有的字符串都需要入池。
所以采用了半自动管理机制，提供了String.Intern和String.IsInterned接口，交给程序员自己维护内部的池。
所以采用了半自动管理机制，提供了字符串。实习生和字符串。

- 常量(字面值）字符串以及使用"+"运算符连接的字面值，会被自动加入到缓存池中，以保证相同内容常量值只存在一个实例。

## String的Intern和IsInterned

- 字符串调用string.Intern()时，如果自身字面值对应对象已经在池中，则返回指向池内对象的新引用。如果自身字面值不在池中，则将自身加入池中返回自身引用。

- 字符串调用string.IsInterned()时，如果自身字面值对应对象已在池中，则返回池内对象引用，如果自身字面值不在池中，则返回null

## 如何产生GC

- string类提供的拼接，格式化等方法，虽然速度快效率高，但是每次调用时内部都会新建临时变量堆积在内存中，很容易产生GC

- 字符串就是char[]， 长短发生变化必然要重新分配长度，以及拷贝之前的部分，产生GC

- 字符串+=拼接会产生装箱，产生GC

- Append传入值类型数据，会调用ToString方法，需要new string 然后把char[]拷进去，产生堆内存

- new StringBuidler 会产生堆内存

- string.format内部使用的就是StringBuidler，但是new StringBuidler、Append、ToString都会产生堆内存。

## StringBuilder Format()
StringBuilder 上现有的 AppendFormat() 方法会生成大量垃圾。
那么`.NET` 以什么方式产生垃圾呢？嗯，参数类型是'object'，由于整数和浮点数经常与 Format() 一起使用，所以你会得到值类型的装箱和拆箱。
还有在“String::ToCharArray()”和“String::CtorCharArrayStartLength()”中进行的临时分配
如果你使用三个以上的参数，也会有更多的垃圾；为此，它需要创建一个临时数组。
看看 StringBuilder 提供的Format()

```Markdown
        public StringBuilder AppendFormat(IFormatProvider provider, string format, object arg0);
        public StringBuilder AppendFormat(IFormatProvider provider, string format, object arg0, object arg1);
        public StringBuilder AppendFormat(IFormatProvider provider, string format, params object[] args);
        public StringBuilder AppendFormat(string format, object arg0);
        public StringBuilder AppendFormat(string format, object arg0, object arg1);
        public StringBuilder AppendFormat(string format, object arg0, object arg1, object arg2);
        public StringBuilder AppendFormat(string format, params object[] args);
        public StringBuilder AppendFormat(IFormatProvider provider, string format, object arg0, object arg1, object arg2);
```

这些中的每一个都使用object参数。如果我想传递整数和浮点数，object参数将受制于值类型的装箱和拆箱。还注意到另外一个数组参数`params`关键词也会造成临时分配，即使不使用object类型。

'params' 关键字实质上将您可能指定的任何参数转换为数组。在这种情况下，整数数组被分配为临时数组，然后被销毁。使用类类型时也会发生同样的事情，而不仅仅是值类型。

## StringBuidler 与 ToString()
C# 中的 StringBuilder 类是处理字符串操作的首选方式,如果你专门使用string类型，则很可能在运行时生成垃圾，但你可以完全避免它。

例如，避免使用“+”运算符连接字符串，c#中字符串是不可变的，这意味着它们不能被修改。如果你执行这种操作，您的结果字符串将被分配在堆上，使用这种代码很容易产生大量垃圾。

StringBuilder 允许您使用可变字符串。它提供将其他字符串和其他类型
连接到可变字符串的功能。StringBuilder 的许多功能是完全无垃圾的。

## ToString()
调用 ToString() 会产生垃圾吗？

以下是源码：

```Markdown
public override String ToString() {
    String currentString =  m_StringValue;
    IntPtr currentThread = m_currentThread;
    if (currentThread != Thread.InternalGetCurrentThread()) {
        return String.InternalCopy(currentString);
    }
    if ((2 *  currentString.Length) &lt; currentString.ArrayLength) {
        return String.InternalCopy(currentString);
    }
    currentString.ClearPostNullChar();
    m_currentThread = IntPtr.Zero;
    return  currentString;
}
```
InternalCopy() 是在堆上分配字符串副本并返回它的函数。该函数实际上可以返回内部字符串而无需进行任何复制。特别是第一次调用时，如果字符串使用了至少 50% 的 StringBuilder 容量。将 'm_currentThread' 成员设置为零实际上意味着下次在 StringBuilder 上执行任何类型的可变操作时；无论是清除字符串，还是附加到它。StringBuilder 将在堆上分配一个新字符串。StringBuilder 的每个可变方法都会检查“m_currentThread”并确保 StringBuilder 拥有特定的字符串。ToString() 的工作方式是有效地放弃该字符串的所有权，并将在下一个可变操作中从一个新的字符串开始。

这个线程注释背后的原因，以及保持 ToString() 返回的 String 完全不可变，是为了保持 StringBuilder 线程安全。

进行复制的另一个原因是用户对内存的使用效率低下。如果我们只使用 StringBuilder 容量的一小部分，.NET 作者认为返回副本而不是引用完整的大字符串更有效。

如果我想重新使用 StringBuilder 对象，我可能会避免在第一次调用时生成字符串的副本，但是如果我重用 StringBuilder，当我调用它的下一个可变方法时，它会在堆上为我分配一个新字符串。
## StringBuilder 到 String 没有垃圾
可以做到的是直接访问 StringBuilder 使用的字符串，下面的一个方法它做了一些.net作者不希望你做的事情
```Markdown
string my_string = (string)my_stringbuilder.GetType().GetField(
    "m_StringValue",
    System.Reflection.BindingFlags.NonPublic |
    System.Reflection.BindingFlags.Instance ).GetValue( my_stringbuilder );
```
该方法并不太好，总归可以去实现。反射再性能方便不太好，所以建议只为您将使用的StringBuilder抓取一次内部字符串。将它作为类的成员或其他东西藏起来，然后您就可以将字符串传递给您希望的任何函数。现在我们不再担心 ToString() 为我们生成垃圾
示例代码：
```Markdown
StringBuilder my_stringbuilder = new StringBuilder( 32, 32 );
string my_string = (string)my_stringbuilder.GetType().GetField(
	"m_StringValue", BindingFlags.NonPublic | BindingFlags.Instance )
	.GetValue( my_stringbuilder );

my_stringbuilder.Append( "This " );
my_stringbuilder.Append( "Is " );
my_stringbuilder.Append( "A " );
my_stringbuilder.Append( "Test!" );

// my string will be "This Is A Test!"
Console.Write( my_string );

// This second append would have resulted in a new heap allocation
// if I'd used ToString() above.
my_stringbuilder.Append( " Yay!" );
Console.Write( my_string );
```
建议使用两个整数参数构造函数创建 StringBuilder 对象。除了预分配字符串外，您还将设置容量上限：
```Markdown
// Creates an empty StringBuilder with a minimum capacity of capacity
// and a maximum capacity of maxCapacity.
public StringBuilder(int capacity, int maxCapacity)
```
这意味着它的大小永远不会增加，也不会在最初构建后进行更多的堆分配。如果发生这种情况，您抓取的字符串对象将不再链接到 StringBuilder。您需要重做反射调用以获取 StringBuilder 拥有的新字符串。

因此，通过使用该构造函数，这意味着如果您获取内部字符串，则可以保证永远保留对 StringBuilder 可变字符串的引用。不过，这里的一个关键例外是，如果您在另一个线程上使用 StringBuilder。在这种情况下，您将遇到与以前相同的问题。

## 如何优化
1. 例：

    ``` 
	int a =100;
	string b = a+"";
    ```
    `a+""`会产生一次装箱操作，应该改成`a.ToString();`

2. 少用string.format，提前缓存共享的StringBuidler对象，避免用的时候产生堆内存。
3. 提前定义好 0 – 9 之间的字符数组，如果传入值类型数据，从高位依次除以10算出每一位的数，然后再去预先声明的0-9字符数组中找对应的char，这样就就不会产生装箱GC了。
4. 如果可以提前确定字符串的长度，可以提前声明一个固定长度的StringBuilder，通过反射取出来内部的_str，这样就可以避免最后的ToString产生的堆内存了。由于_str内容可能无法擦掉之前的所以需要调用GarbageFreeClear();方法。

## 使用 StringBuilder 时避免垃圾
[StringBuilderExtFormat.zip](StringBuilderExtFormat.zip)

无GC代码示例：
```Markdown
using System;
using System.Text;
using UnityEngine;
 
public static class StringBuilderExtensions
{
    // These digits are here in a static array to support hex with simple, easily-understandable code. 
    // Since A-Z don't sit next to 0-9 in the ascii table.
    private static readonly char[] ms_digits = new char[] { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };
 
    private static readonly uint ms_default_decimal_places = 5; //< Matches standard .NET formatting dp's
    private static readonly char ms_default_pad_char = '0';
 
    //! Convert a given unsigned integer value to a string and concatenate onto the stringbuilder. Any base value allowed.
    public static StringBuilder Concat(this StringBuilder string_builder, uint uint_val, uint pad_amount, char pad_char, uint base_val)
    {
        Debug.Assert(pad_amount >= 0);
        Debug.Assert(base_val > 0 && base_val <= 16);
 
        // Calculate length of integer when written out
        uint length = 0;
        uint length_calc = uint_val;
 
        do
        {
            length_calc /= base_val;
            length++;
        }
        while (length_calc > 0);
 
        // Pad out space for writing.
        string_builder.Append(pad_char, (int)Math.Max(pad_amount, length));
 
        int strpos = string_builder.Length;
 
        // We're writing backwards, one character at a time.
        while (length > 0)
        {
            strpos--;
 
            // Lookup from static char array, to cover hex values too
            string_builder[strpos] = ms_digits[uint_val % base_val];
 
            uint_val /= base_val;
            length--;
        }
 
        return string_builder;
    }
 
    //! Convert a given unsigned integer value to a string and concatenate onto the stringbuilder. Assume no padding and base ten.
    public static StringBuilder Concat(this StringBuilder string_builder, uint uint_val)
    {
        string_builder.Concat(uint_val, 0, ms_default_pad_char, 10);
        return string_builder;
    }
 
    //! Convert a given unsigned integer value to a string and concatenate onto the stringbuilder. Assume base ten.
    public static StringBuilder Concat(this StringBuilder string_builder, uint uint_val, uint pad_amount)
    {
        string_builder.Concat(uint_val, pad_amount, ms_default_pad_char, 10);
        return string_builder;
    }
 
    //! Convert a given unsigned integer value to a string and concatenate onto the stringbuilder. Assume base ten.
    public static StringBuilder Concat(this StringBuilder string_builder, uint uint_val, uint pad_amount, char pad_char)
    {
        string_builder.Concat(uint_val, pad_amount, pad_char, 10);
        return string_builder;
    }
 
    //! Convert a given signed integer value to a string and concatenate onto the stringbuilder. Any base value allowed.
    public static StringBuilder Concat(this StringBuilder string_builder, int int_val, uint pad_amount, char pad_char, uint base_val)
    {
        Debug.Assert(pad_amount >= 0);
        Debug.Assert(base_val > 0 && base_val <= 16);
 
        // Deal with negative numbers
        if (int_val < 0)
        {
            string_builder.Append('-');
            uint uint_val = uint.MaxValue - ((uint)int_val) + 1; //< This is to deal with Int32.MinValue
            string_builder.Concat(uint_val, pad_amount, pad_char, base_val);
        }
        else
        {
            string_builder.Concat((uint)int_val, pad_amount, pad_char, base_val);
        }
 
        return string_builder;
    }
 
    //! Convert a given signed integer value to a string and concatenate onto the stringbuilder. Assume no padding and base ten.
    public static StringBuilder Concat(this StringBuilder string_builder, int int_val)
    {
        string_builder.Concat(int_val, 0, ms_default_pad_char, 10);
        return string_builder;
    }
 
    //! Convert a given signed integer value to a string and concatenate onto the stringbuilder. Assume base ten.
    public static StringBuilder Concat(this StringBuilder string_builder, int int_val, uint pad_amount)
    {
        string_builder.Concat(int_val, pad_amount, ms_default_pad_char, 10);
        return string_builder;
    }
 
    //! Convert a given signed integer value to a string and concatenate onto the stringbuilder. Assume base ten.
    public static StringBuilder Concat(this StringBuilder string_builder, int int_val, uint pad_amount, char pad_char)
    {
        string_builder.Concat(int_val, pad_amount, pad_char, 10);
        return string_builder;
    }
 
    //! Convert a given float value to a string and concatenate onto the stringbuilder
    public static StringBuilder Concat(this StringBuilder string_builder, float float_val, uint decimal_places, uint pad_amount, char pad_char)
    {
        Debug.Assert(pad_amount >= 0);
 
        if (decimal_places == 0)
        {
            // No decimal places, just round up and print it as an int
 
            // Agh, Math.Floor() just works on doubles/decimals. Don't want to cast! Let's do this the old-fashioned way.
            int int_val;
            if (float_val >= 0.0f)
            {
                // Round up
                int_val = (int)(float_val + 0.5f);
            }
            else
            {
                // Round down for negative numbers
                int_val = (int)(float_val - 0.5f);
            }
 
            string_builder.Concat(int_val, pad_amount, pad_char, 10);
        }
        else
        {
            int int_part = (int)float_val;
 
            // First part is easy, just cast to an integer
            string_builder.Concat(int_part, pad_amount, pad_char, 10);
 
            // Decimal point
            string_builder.Append('.');
 
            // Work out remainder we need to print after the d.p.
            float remainder = Math.Abs(float_val - int_part);
 
            // Multiply up to become an int that we can print
            do
            {
                remainder *= 10;
                decimal_places--;
            }
            while (decimal_places > 0);
 
            // Round up. It's guaranteed to be a positive number, so no extra work required here.
            remainder += 0.5f;
 
            // All done, print that as an int!
            string_builder.Concat((uint)remainder, 0, '0', 10);
        }
        return string_builder;
    }
 
    //! Convert a given float value to a string and concatenate onto the stringbuilder. Assumes five decimal places, and no padding.
    public static StringBuilder Concat(this StringBuilder string_builder, float float_val)
    {
        string_builder.Concat(float_val, ms_default_decimal_places, 0, ms_default_pad_char);
        return string_builder;
    }
 
    //! Convert a given float value to a string and concatenate onto the stringbuilder. Assumes no padding.
    public static StringBuilder Concat(this StringBuilder string_builder, float float_val, uint decimal_places)
    {
        string_builder.Concat(float_val, decimal_places, 0, ms_default_pad_char);
        return string_builder;
    }
 
    //! Convert a given float value to a string and concatenate onto the stringbuilder.
    public static StringBuilder Concat(this StringBuilder string_builder, float float_val, uint decimal_places, uint pad_amount)
    {
        string_builder.Concat(float_val, decimal_places, pad_amount, ms_default_pad_char);
        return string_builder;
    }
 
    //! Concatenate a formatted string with arguments
    public static StringBuilder ConcatFormat<A>(this StringBuilder string_builder, String format_string, A arg1)
        where A : IConvertible
    {
        return string_builder.ConcatFormat<A, int, int, int>(format_string, arg1, 0, 0, 0);
    }
 
    //! Concatenate a formatted string with arguments
    public static StringBuilder ConcatFormat<A, B>(this StringBuilder string_builder, String format_string, A arg1, B arg2)
        where A : IConvertible
        where B : IConvertible
    {
        return string_builder.ConcatFormat<A, B, int, int>(format_string, arg1, arg2, 0, 0);
    }
 
    //! Concatenate a formatted string with arguments
    public static StringBuilder ConcatFormat<A, B, C>(this StringBuilder string_builder, String format_string, A arg1, B arg2, C arg3)
        where A : IConvertible
        where B : IConvertible
        where C : IConvertible
    {
        return string_builder.ConcatFormat<A, B, C, int>(format_string, arg1, arg2, arg3, 0);
    }
 
    //! Concatenate a formatted string with arguments
    public static StringBuilder ConcatFormat<A, B, C, D>(this StringBuilder string_builder, String format_string, A arg1, B arg2, C arg3, D arg4)
        where A : IConvertible
        where B : IConvertible
        where C : IConvertible
        where D : IConvertible
    {
        int verbatim_range_start = 0;
 
        for (int index = 0; index < format_string.Length; index++)
        {
            if (format_string[index] == '{')
            {
                // Formatting bit now, so make sure the last block of the string is written out verbatim.
                if (verbatim_range_start < index)
                {
                    // Write out unformatted string portion
                    string_builder.Append(format_string, verbatim_range_start, index - verbatim_range_start);
                }
 
                uint base_value = 10;
                uint padding = 0;
                uint decimal_places = 5; // Default decimal places in .NET libs
 
                index++;
                char format_char = format_string[index];
                if (format_char == '{')
                {
                    string_builder.Append('{');
                    index++;
                }
                else
                {
                    index++;
 
                    if (format_string[index] == ':')
                    {
                        // Extra formatting. This is a crude first pass proof-of-concept. It's not meant to cover
                        // comprehensively what the .NET standard library Format() can do.
                        index++;
 
                        // Deal with padding
                        while (format_string[index] == '0')
                        {
                            index++;
                            padding++;
                        }
 
                        if (format_string[index] == 'X')
                        {
                            index++;
 
                            // Print in hex
                            base_value = 16;
 
                            // Specify amount of padding ( "{0:X8}" for example pads hex to eight characters
                            if ((format_string[index] >= '0') && (format_string[index] <= '9'))
                            {
                                padding = (uint)(format_string[index] - '0');
                                index++;
                            }
                        }
                        else if (format_string[index] == '.')
                        {
                            index++;
 
                            // Specify number of decimal places
                            decimal_places = 0;
 
                            while (format_string[index] == '0')
                            {
                                index++;
                                decimal_places++;
                            }
                        }
                    }
 
 
                    // Scan through to end bracket
                    while (format_string[index] != '}')
                    {
                        index++;
                    }
 
                    // Have any extended settings now, so just print out the particular argument they wanted
                    switch (format_char)
                    {
                        case '0': string_builder.ConcatFormatValue<A>(arg1, padding, base_value, decimal_places); break;
                        case '1': string_builder.ConcatFormatValue<B>(arg2, padding, base_value, decimal_places); break;
                        case '2': string_builder.ConcatFormatValue<C>(arg3, padding, base_value, decimal_places); break;
                        case '3': string_builder.ConcatFormatValue<D>(arg4, padding, base_value, decimal_places); break;
                        default: Debug.Assert(false, "Invalid parameter index"); break;
                    }
                }
 
                // Update the verbatim range, start of a new section now
                verbatim_range_start = (index + 1);
            }
        }
 
        // Anything verbatim to write out?
        if (verbatim_range_start < format_string.Length)
        {
            // Write out unformatted string portion
            string_builder.Append(format_string, verbatim_range_start, format_string.Length - verbatim_range_start);
        }
 
        return string_builder;
    }
 
    //! The worker method. This does a garbage-free conversion of a generic type, and uses the garbage-free Concat() to add to the stringbuilder
    private static void ConcatFormatValue<T>(this StringBuilder string_builder, T arg, uint padding, uint base_value, uint decimal_places) where T : IConvertible
    {
        switch (arg.GetTypeCode())
        {
            case System.TypeCode.UInt32:
                {
                    string_builder.Concat(arg.ToUInt32(System.Globalization.NumberFormatInfo.CurrentInfo), padding, '0', base_value);
                    break;
                }
 
            case System.TypeCode.Int32:
                {
                    string_builder.Concat(arg.ToInt32(System.Globalization.NumberFormatInfo.CurrentInfo), padding, '0', base_value);
                    break;
                }
 
            case System.TypeCode.Single:
                {
                    string_builder.Concat(arg.ToSingle(System.Globalization.NumberFormatInfo.CurrentInfo), decimal_places, padding, '0');
                    break;
                }
 
            case System.TypeCode.String:
                {
                    string_builder.Append(Convert.ToString(arg));
                    break;
                }
 
            default:
                {
                    Debug.Assert(false, "Unknown parameter type");
                    break;
                }
        }
    }
 
    public static void Empty(this StringBuilder string_builder)
    {
        string_builder.Remove(0, string_builder.Length);
    }
 
 
    public static string GetGarbageFreeString(this StringBuilder string_builder)
    {
        return (string)string_builder.GetType().GetField("_str", System.Reflection.BindingFlags.NonPublic | System.Reflection.BindingFlags.Instance).GetValue(string_builder);
    }
 
    public static void GarbageFreeClear(this StringBuilder string_builder)
    {
        string_builder.Length = 0;
        string_builder.Append(' ', string_builder.Capacity);
        string_builder.Length = 0;
    }
}
public static class StrExt
{
    //只允许拼接50个字符串
    private static StringBuilder s_StringBuilder = new StringBuilder(50, 50);
 
    public static string Format<A>(String format_string, A arg1)
            where A : IConvertible
    {
        s_StringBuilder.Empty();
        return s_StringBuilder.ConcatFormat<A, int, int, int>(format_string, arg1, 0, 0, 0).ToString();
    }
 
    public static string Format<A, B>(String format_string, A arg1, B arg2)
        where A : IConvertible
        where B : IConvertible
    {
        s_StringBuilder.Empty();
        return s_StringBuilder.ConcatFormat<A, B, int, int>(format_string, arg1, arg2, 0, 0).ToString();
    }
 
    public static string Format<A, B, C>(String format_string, A arg1, B arg2, C arg3)
        where A : IConvertible
        where B : IConvertible
        where C : IConvertible
    {
        s_StringBuilder.Empty();
        return s_StringBuilder.ConcatFormat<A, B, C, int>(format_string, arg1, arg2, arg3, 0).ToString();
    }
 
    public static string Format<A, B, C, D>(String format_string, A arg1, B arg2, C arg3, D arg4)
        where A : IConvertible
        where B : IConvertible
        where C : IConvertible
        where D : IConvertible
    {
        s_StringBuilder.Empty();
        return s_StringBuilder.ConcatFormat<A, B, C, D>(format_string, arg1, arg2, arg3, arg4).ToString();
    }
}
 
```



参考文章[StringBuilder and String Garbage](https://www.gavpugh.com/xnac-articles/)

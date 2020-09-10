---
keyword: string functions
---

# String functions

This topic describes the string functions provided by MaxCompute, which can be used in SQL statements.

## CHAR\_MATCHCOUNT

-   Syntax

    ```
    BIGINT CHAR_MATCHCOUNT(STRING str1, STRING str2)
    ```

-   Description

    This function returns the number of characters that belong to str1 and appear in str2.

-   Parameters

    str1 and str2: the valid UTF-8 strings. The function returns a negative value if invalid characters are found during the comparison of the two strings.

-   Return value

    Returns a value of the BIGINT type. NULL is returned if an input parameter is NULL.

-   Examples

    ```
    CHAR_MATCHCOUNT('abd','aabc') = 2
    -- The a and b characters in str1 appear in str2.
    ```


## CHR

-   Syntax

    ```
    STRING CHR(BIGINT ascii)
    ```

-   Description

    This function converts a specified ASCII code to the corresponding character.

-   Parameters

    ascii: a value of the BIGINT type, which indicates the ASCII code. Value range: 0 to 255. An exception is thrown if the input value is out of this range. The input value is implicitly converted to a value of the BIGINT type before computing if it is of the STRING, DOUBLE, or DECIMAL type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.


## CONCAT

-   Syntax

    ```
    STRING CONCAT(STRING a, STRING b...)
    ```

-   Description

    This function concatenates all specified strings and returns the final string.

-   Parameters

    The values of all parameters are of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if no input parameters are present or an input parameter is NULL.

-   Examples

    ```
    CONCAT('ab','c') = 'abc'
    CONCAT() = NULL
    CONCAT('a', null, 'b') = NULL
    ```


## GET\_JSON\_OBJECT

-   Syntax

    ```
    STRING GET_JSON_OBJECT(STRING json,STRING path)
    ```

-   Description

    This function extracts the specified string from a standard JSON string based on `path`. The original data is read each time this function is called. Therefore, repeated calls may waste system resources and increase your cost. To avoid repeated calls, you can use the `GET_JSON_OBJECT` function together with User-Defined Table-Generating Functions \(UDTFs\). For more information, see [Convert JSON log data by using MaxCompute built-in functions and UDTFs](https://yq.aliyun.com/articles/627758).

-   Parameters
    -   `json`: the string in the standard `JSON` format.
    -   `path`: the string that starts with the dollar sign \(`$`\), which describes the path of a JSON object in the JSON string.```` For more information about `path`, see [LanguageManual UDF](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF).
        -   `$`: indicates the root node.
        -   `.`: indicates a child node.
        -   `[]`: `[number]` indicates the array subscript. The array is represented in the format of `key[sub1][sub2][sub3]……`.
        -   `*`: indicates the wildcard for `[]`, which returns the entire array. An asterisk \(`*`\) cannot be escaped.
-   Return value
    -   Returns `NULL` if `json` is empty or invalid.
    -   Returns the corresponding string if `json` is valid and the specified `path` exists.
    -   A key cannot appear in an object twice. For example, `{a:1, a:0}` is not supported. Otherwise, the command may fail to be parsed.
    -   Emoji expressions are not supported.
-   Example 1

    ```
    +----+
    json
    +----+
    {"store":
    {"fruit":[{"weight":8,"type":"apple"},{"weight":9,"type":"pear"}],
    "bicycle":{"price":19.95,"color":"red"}
    },
    "email":"amy@only_for_json_udf_test.net",
    "owner":"amy"
    }
    ```

    You can execute the following statements to extract data from the preceding `JSON` string.

    ```
    odps> SELECT GET_JSON_OBJECT(src_json.json, '$.owner') FROM src_json;
    amy
    odps> SELECT GET_JSON_OBJECT(src_json.json, '$.store.fruit\[0]') FROM src_json;
    {"weight":8,"type":"apple"}
    odps> SELECT GET_JSON_OBJECT(src_json.json, '$.non_exist_key') FROM src_json;
    NULL
    ```

    Example 2

    ```
    GET_JSON_OBJECT('{"array":[["aaaa",1111],["bbbb",2222],["cccc",3333]]}','$.array[1][1]')= "2222"
    GET_JSON_OBJECT('{"aaa":"bbb","ccc":{"ddd":"eee","fff":"ggg","hhh":["h0","h1","h2"]},"iii":"jjj"}','$.ccc.hhh[*]') = "["h0","h1","h2"]"
    GET_JSON_OBJECT('{"aaa":"bbb","ccc":{"ddd":"eee","fff":"ggg","hhh":["h0","h1","h2"]},"iii":"jjj"}','$.ccc.hhh[1]') = "h1"
    ```


## INSTR

-   Syntax

    ```
    BIGINT INSTR(STRING str1, STRING str2[, BIGINT start_position[, BIGINT nth_appearance]])
    ```

-   Description

    This function returns the position of substring `str2` in string `str1`.

-   Parameters
    -   `str1`: a value of the STRING type, which indicates the string to be searched. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.
    -   `str2`: a value of the STRING type, which indicates the substring to be searched for. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.
    -   `start_position`: a value of the BIGINT type. If it is of another data type, an exception is thrown. It indicates the character in `str1` from which the search will start. The start position defaults to the first character, marked as 1.
    -   `nth_appearance`: a value of the BIGINT type, which must be greater than 0. It indicates the nth time that the substring appears in the string, where n is specified by `nth_appearance`. An exception is thrown if the value of `nth_appearance` is of another data type or is no greater than 0.
-   Return value
    -   Returns a value of the BIGINT type.
    -   Returns 0 if `str2` is not found in `str1`.
    -   Returns NULL if an input parameter is NULL.
    -   The matching always succeeds if `str2` is empty. For example, 1 is returned for `instr(‘abc’, ")`.
-   Examples

    ```
    INSTR('Tech on the net', 'e') = 2
    INSTR('Tech on the net', 'e', 1, 1) = 2
    INSTR('Tech on the net', 'e', 1, 2) = 11
    INSTR('Tech on the net', 'e', 1, 3) = 14
    ```


## IS\_ENCODING

-   Syntax

    ```
    BOOLEAN IS_ENCODING(STRING str, STRING from_encoding, STRING to_encoding)
    ```

-   Description

    This function determines whether the input string `str` can be converted from the character set specified by `from_encoding` to the character set specified by `to_encoding`. It can be used to determine whether the input string is garbled. Generally, `from_encoding` is set to utf-8 and `to_encoding` is set to gbk.

-   Parameters
    -   `str`: a value of the STRING type. An empty string can belong to any character set.
    -   `from_encoding` and `to_encoding`: the source and target character sets. The values are of the STRING type.
    -   NULL is returned if an input parameter is NULL.
-   Return value

    Returns a value of the BOOLEAN type. If `str` can be converted, true is returned. Otherwise, false is returned.


## KEYVALUE

-   Syntax

    ```
    KEYVALUE(STRING srcStr,STRING split1,STRING split2, STRING key)
    KEYVALUE(STRING srcStr,STRING key) //split1 = ";",split2 = ":"
    ```

-   Description

    This function splits the source string `srcStr` into key-value pairs by `split1`, separates key-value pairs by `split2`, and then returns the value of the specified key.``

-   Parameters
    -   `srcStr`: the source string to be split.
    -   `key`: the key corresponds to the value returned after the source string is split by `split1` and `split2` in sequence. This parameter is of the STRING type.``
    -   `split1` and `split2`: the strings that are used as delimiters to split the source string. If you do not specify these two parameters, `split1` defaults to `;` and `split2` defaults to `:`. If a key-value pair that is obtained after the source string is split by `split1` contains multiple delimiters specified by `split2`, the returned result is undefined.
-   Return value
    -   Returns a value of the STRING type.
    -   Returns NULL if `split1` or `split2` is NULL.
    -   Returns NULL if `scrStr` and `key` are NULL or if no `key` matches.
    -   Returns the value corresponding to the first matched `key` if multiple key-value pairs match.
-   Example 1

    ```
    KEYVALUE('0:1\;1:2', 1) = '2'
    ```

    The source string is `"0:1\;1:2"`. `split1` defaults to `;` and `split2` defaults to `:` because they are not specified.````

    After the source string is split by `split1`, the key-value pairs are `0:1\,1:2`. After the key-value pairs are separated by `split2`, the key-value pairs change to the following form:

    ```
    0 1/  
    1 2
    ```

    The value 2 corresponding to key `1` is returned.

    Example 2

    ```
    KEYVALUE("\;decreaseStore:1\;xcard:1\;isB2C:1\;tf:21910\;cart:1\;shipping:2\;pf:0\;market:shoes\;instPayAmount:0\;","\;",":","tf") = "21910" value:21910
    ```

    The source string is `"\;decreaseStore:1\;xcard:1\;isB2C:1\;tf:21910\;cart:1\;shipping:2\;pf:0\;market:shoes\;instPayAmount:0\;"`. After the source string is split by `\;` specified by split1, the following key-value pairs are generated:

    ```
    decreaseStore:1,xcard:1,isB2C:1,tf:21910,cart:1,shipping:2,pf:0,market:shoes,instPayAmount:0 
    ```

    After the key-value pairs are split by `:` specified by split2, the key-value pairs change to the following form:

    ```
    decreaseStore 1  
    xcard 1  
    isB2C 1  
    tf 21910  
    cart 1  
    shipping 2  
    pf 0  
    market shoes  
    instPayAmount 0
    ```

    The value 21910 corresponding to key `tf` is returned.


## LENGTH

-   Syntax

    ```
    BIGINT LENGTH(STRING str)
    ```

-   Description

    This function returns the length of the string `str`.

-   Parameters
    -   `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.
    -   Returns a value of the BIGINT type. NULL is returned if the input parameter is NULL. -1 is returned if the input parameter is not encoded in UTF-8.
-   Examples

    ```
    LENGTH('hi! CN') = 6
    ```


## LENGTHB

-   Syntax

    ```
    BIGINT LENGTHB(STRING str)
    ```

-   Description

    This function returns the length of the string `str` in bytes.

-   Parameters
    -   `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.
    -   Returns a value of the BIGINT type. NULL is returned if the input parameter is NULL.
-   Examples

    ```
    LENGTHB('hi! China!') = 10
    ```


## MD5

-   Syntax

    ```
    STRING MD5(STRING value)
    ```

-   Description

    This function returns the MD5 value of the input string `value`.

-   Parameters

    `value`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.


## REGEXP\_EXTRACT

-   Syntax

    ```
    STRING REGEXP_EXTRACT(STRING source, STRING pattern[, BIGINT occurrence])
    ```

-   Description

    This function splits the `source` string based on a given `pattern` and returns the characters in the `group` at the nth occurrence, where n is specified by `occurrence`.

-   Parameters

    -   `source`: the string to split and search.
    -   `pattern`: a constant of the STRING type. An exception is thrown if `pattern` is an empty string or no `group` is specified in `pattern`.
    -   `occurrence`: a constant of the BIGINT type, which must be no less than 0. Otherwise, an exception is thrown. If you do not specify this parameter, it defaults to 1, indicating that characters in the first `group` are returned. If you set `occurrence` to 0, all substrings that match the regular expression specified by `pattern` are returned.
    **Note:** Data is stored in the UTF-8 format. Chinese characters can be represented in hexadecimal. They are encoded in the range of \[\\x\{4e00\}-\\x\{9fa5\}\].

-   Return value

    Returns a value of the STRING type. Returns NULL if an input parameter is NULL.


## REGEXP\_INSTR

-   Syntax

    ```
    BIGINT REGEXP_INSTR(STRING source, STRING pattern[,
    BIGINT start_position[, BIGINT nth_occurrence[, BIGINT return_option]]])
    ```

-   Description

    This function returns the start or end position of the substring that matches `pattern` at the nth occurrence, where n is specified by `nth_occurrence`, in the `source` string from the start position specified by `start_position`.

-   Parameters
    -   `source`: a value of the STRING type, which indicates the string to be searched.
    -   `pattern`: a constant of the STRING type. An exception is thrown if `pattern` is an empty string.
    -   `start_position`: a constant of the BIGINT type, which indicates the start position of a search. If you do not specify this parameter, its value defaults to 1. An exception is thrown if its value is of another data type or is no greater than 0.
    -   `nth_occurrence`: a constant of the BIGINT type. If you do not specify this parameter, its value defaults to 1, indicating the first occurrence of the matched substring. An exception is thrown if its value is of another data type or is no greater than 0.
    -   `return_option`: a constant of the BIGINT type, which can be 0 or 1. An exception is thrown if its value is of another data type or is out of the value range. 0 indicates that the start position of the matched substring is returned, and 1 indicates that the end position of the matched substring is returned.
-   Return value

    Returns a value of the BIGINT type. It is the start or end position of the matched substring in the `source` string based on the type specified by `return_option`. NULL is returned if an input parameter is NULL.

-   Examples

    ```
    REGEXP_INSTR("i love www.taobao.com", "o[[:alpha:]]{1}", 3, 2) = 14
    ```


## REGEXP\_REPLACE

-   Syntax

    ```
    STRING REGEXP_REPLACE(STRING source, STRING pattern, STRING replace_string[, BIGINT occurrence])
    ```

-   Description

    This function substitutes the specified string `replace_string` for the substring that matches a given `pattern` at the nth occurrence, where n is specified by `occurrence`, in the `source` string, and returns the result.

-   Parameters
    -   `source`: the string to be replaced.
    -   `pattern`: a constant of the STRING type, which indicates the pattern to be matched. An exception is thrown if `pattern` is an empty string.
    -   `replace_string`: a value of the STRING type, which is a string that is substituted for the substring matching the given `pattern`.
    -   `occurrence`: a constant of the BIGINT type, which must be no less than 0. It indicates that `replace_string` is substituted for the matching substring at the nth occurrence. The value 0 indicates that all matching substrings are replaced. An exception is thrown if its value is of another type or is less than 0. 0 is assumed by default.
-   Return value
    -   Returns a value of the STRING type. If the string to be replaced does not exist, the replacement does not occur.
    -   Returns NULL if an input parameter is NULL.
    -   Returns NULL if `replace_string` is NULL and certain substrings match the given `pattern`.
    -   Returns the original string if `replace_string` is NULL but no substring matches the given `pattern`.
-   Examples

    ```
    REGEXP_REPLACE("123.456.7890", "([[:digit:]]{3})\\.([[:digit:]]{3})\\.([[:digit:]]{4})",
    "(\\1)\\2-\\3", 0) = "(123)456-7890"
    REGEXP_REPLACE("abcd", "(.)", "\\1 ", 0) = "a b c d "
    REGEXP_REPLACE("abcd", "(.)", "\\1 ", 1) = "a bcd"
    REGEXP_REPLACE("abcd", "(.)", "\\2", 1) = "abcd"
    -- Only one group is defined in the pattern and the referenced second group does not exist.
    -- Try to avoid this action. The result of referencing a nonexistent group is undefined.
    REGEXP_REPLACE("abcd", "(. *)(.)$", "\\2", 0) = "d"
    REGEXP_REPLACE("abcd", "a", "\\1", 0) = "bcd"
    -- No group is defined in the pattern, so "\1" references a nonexistent group.
    -- Try to avoid this action. The result of referencing a nonexistent group is undefined.
    ```


## REGEXP\_SUBSTR

-   Syntax

    ```
    STRING REGEXP_SUBSTR(STRING source, STRING pattern[, BIGINT start_position[, BIGINT nth_occurrence]])
    ```

-   Description

    This function returns the string that matches a given `pattern` at the nth occurrence, where n is specified by `nth_occurrence`, in the `source` string from the start position specified by `start_position`.

-   Parameters
    -   `source`: a value of the STRING type, which indicates the string to be searched.
    -   `pattern`: a constant of the STRING type, which indicates the pattern to be matched. An exception is thrown if `pattern` is an empty string.
    -   `start_position`: a constant of the BIGINT type, which must be greater than 0. An exception is thrown if its value is of another data type or is no greater than 0. If you do not specify this parameter, its value defaults to 1, indicating that the match starts from the first character of the `source`.
    -   `nth_occurrence`: a constant of the BIGINT type, which must be greater than 0. An exception is thrown if its value is of another data type or is no greater than 0. If you do not specify this parameter, its value defaults to 1, indicating that the first matched substring is returned.
-   Return value

    Returns a value of the STRING type. NULL is returned if an input parameter is NULL or no substrings match.

-   Examples

    ```
    REGEXP_SUBSTR("I love aliyun very much", "a[[:alpha:]]{5}") = "aliyun"
    REGEXP_SUBSTR('I have 2 apples and 100 bucks!', '[[:blank:]][[:alnum:]]*', 1, 1) = " have"
    REGEXP_SUBSTR('I have 2 apples and 100 bucks!', '[[:blank:]][[:alnum:]]*', 1, 2) = " 2"
    ```


## REGEXP\_COUNT

-   Syntax

    ```
    BIGINT REGEXP_COUNT(STRING source, STRING pattern[, BIGINT start_position])
    ```

-   Description

    This function returns the number of times a substring matches a given `pattern` in the `source` string from the start position specified by `start_position`.

-   Parameters
    -   `source`: a value of the STRING type, which indicates the string to be searched. An exception is thrown if its value is of another data type.
    -   `pattern`: a constant of the STRING type, which indicates the pattern to be matched. An exception is thrown if `pattern` is an empty string or is of another data type.
    -   `start_position`: a constant of the BIGINT type, which must be greater than 0. An exception is thrown if its value is of another data type or is no greater than 0. If you do not specify this parameter, its value defaults to 1, indicating that the match starts from the first character of the `source` string.
-   Return value

    Returns a value of the BIGINT type. 0 is returned if no substring is matched. Null is returned if an input parameter is NULL.

-   Examples

    ```
    REGEXP_COUNT('abababc', 'a.c') = 1
    REGEXP_COUNT('abcde', '[[:alpha:]]{2}', 3) = 1
    ```


## SPLIT\_PART

-   Syntax

    ```
    STRING SPLIT_PART(STRING str, STRING separator, BIGINT start[, BIGINT end])
    ```

-   Description

    This function uses a delimiter specified by `separator` to split the string `str`, and returns a substring that starts from the character specified by `start` and ends with the character specified by `end`.

-   Parameters
    -   `str`: a value of the STRING type, which indicates a string to be split. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.
    -   `separator`: a constant of the STRING type, which is a separator used for splitting. It can be a character or string. If it is of another data type, an exception is thrown.
    -   `start`: a constant of the BIGINT type, which must be greater than 0. An exception is thrown if its value is not a constant or is of another data type. It indicates the start number of the segment to be returned, starting from 1. If `end` is not specified, the segment specified by `start` is returned.
    -   `end`: a constant of the BIGINT type, which must be no less than `start`. It indicates the end number of the segment to be returned. An exception is thrown if its value is not a constant or is of another data type. If end is not specified, the last segment is returned.
-   Return value
    -   Returns a value of the STRING type.
    -   Returns an empty string if `start` is set to a value greater than the number of segments, for example, the string has 6 segments but the `start` value is greater than 6.
    -   Returns the entire string `str` if `separator` is absent in the string `str` and `start` is set to 1. An empty string is returned if `str` is an empty string.
    -   Returns the original string `str` if `separator` is an empty string.
    -   Returns the string between start and the last segment if `end` is set to a value greater than the number of segments.
    -   Returns NULL if an input parameter is NULL.
-   Examples

    ```
    SPLIT_PART('a,b,c,d', ',', 1) = 'a'
    SPLIT_PART('a,b,c,d', ',', 1, 2) = 'a,b'
    SPLIT_PART('a,b,c,d', ',', 10) = ''
    ```


## SUBSTR

-   Syntax

    ```
    STRING SUBSTR(STRING str, BIGINT start_position[, BIGINT length])
    ```

-   Description

    This function returns a substring that starts from `start_position` in `str` and has a length specified by `length`.

-   Parameters
    -   `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.
    -   `start_position`: a value of the BIGINT type. 1 is assumed by default. An empty string is returned if `start_position` is 0. If `start_position` is a negative value, the start position is counted backwards from the last character of the string. For example, -1 indicates the last character, -2 indicates the second-to-last character, -3 indicates the third-from-last character, and so on. If the value is of another data type, an exception is thrown.
    -   `length`: a value of the BIGINT type, which indicates the length of the substring. Its value is greater than 0. An exception is thrown if its value of another data type or is no greater than 0.
-   Return value

    Returns a value of the STRING type. NULL is returned if an input parameter is NULL.

    **Note:** If `length` is not specified, the substring from the start position to the end of `str` is returned.

-   Examples

    ```
    SUBSTR("abc", 2) = "bc"
    SUBSTR("abc", 2, 1) = "b"
    SUBSTR("abc",-2,2) = "bc"
    SUBSTR("abc",-3) = "abc"
    ```


## SUBSTRING

-   Syntax

    ```
    STRING SUBSTRING(STRING|BINARY str, INT start_position[, INT length])
    ```

-   Description

    This function returns a substring that starts from `start_position` in `str` and has a length specified by `length`.

-   Parameters
    -   `str`: a value of the STRING or BINARY type. NULL or an error is returned if the input value is of another data type.
    -   `start_position`: a value of the INT type. 1 is assumed by default. An empty string is returned if `start_position` is set to 0. If `start_position` is a negative value, the start position is counted backwards from the last character of the string. For example, -1 indicates the last character, -2 indicates the second-to-last character, -3 indicates the third-from-last character, and so on. If the value is of another data type, an exception is thrown.
    -   `length`: a value of the BIGINT type, which indicates the length of the substring. Its value is greater than 0. An exception is thrown if its value is of another data type or is no greater than 0.
-   Return value

    Returns a value of the STRING type. NULL is returned if an input parameter is NULL.

    **Note:** If `length` is not specified, the substring from the start position to the end of `str` is returned.

-   Examples

    ```
    SUBSTRING('abc', 2) = 'bc'
    SUBSTRING('abc', 2, 1) ='"b'
    SUBSTRING('abc',-2,2) = 'bc'
    SUBSTRING('abc',-3,2) = 'ab'
    SUBSTRING(BIN(2345),2,3) = '001'
    ```


## TOLOWER

-   Syntax

    ```
    STRING TOLOWER(STRING source)
    ```

-   Description

    This function converts the string `source` to a lowercase string.

-   Parameters

    `source`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    tolower("aBcd") = "abcd"
    tolower("ABCd") = "abcd"
    ```


## TOUPPER

-   Syntax

    ```
    STRING TOUPPER(STRING source)
    ```

-   Description

    This function converts the string `source` to an uppercase string.

-   Parameters

    `source`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    toupper("aBcd") = "ABCD"
    toupper("abCd") = "ABCD"
    ```


## TO\_CHAR

-   Syntax

    ```
    STRING TO_CHAR(BOOLEAN value)
    STRING TO_CHAR(BIGINT value)
    STRING TO_CHAR(DOUBLE value)
    STRING TO_CHAR(DECIMAL value)
    ```

-   Description

    This function converts data of the BOOLEAN, BIGINT, DECIMAL, or DOUBLE type to the STRING type.

-   Parameters

    `value`: The input value can be of the BOOLEAN, BIGINT, DECIMAL, or DOUBLE type. An exception is thrown if its value is of another data type.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    TO_CHAR(123) = '123'
    TO_CHAR(true) = 'TRUE'
    TO_CHAR(1.23) = '1.23'
    TO_CHAR(null) = NULL
    ```


## TRIM

-   Syntax

    ```
    STRING TRIM(STRING str)
    ```

-   Description

    This function eliminates the spaces on the left and right sides of the string `str`.

-   Parameters

    `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.


## LTRIM

-   Syntax

    ```
    STRING LTRIM(STRING str)
    ```

-   Description

    This function eliminates the spaces on the left side of the string `str`.

-   Parameters

    `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    DELECT LTRIM(' abc ') FROM dual; 
    -- Returned result:
    +-----+
    | _c0 |
    +-----+
    | abc |
    +-----+
    ```


## RTRIM

-   Syntax

    ```
    STRING RTRIM(STRING str)
    ```

-   Description

    This function eliminates the spaces on the right side of the string `str`.

-   Parameters

    `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    SELECT RTRIM('a abc ') FROM dual; 
    -- Returned result: 
    +-----+
    | _c0 |
    +-----+
    | a abc |
    +-----+
    ```


## REVERSE

-   Syntax

    ```
    STRING REVERSE(STRING str)
    ```

-   Description

    This function returns a string in reverse order.

-   Parameters

    `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    SELECT REVERSE('abcedfg') FROM dual;--Returned result:
    +-----+
    | _c0 |
    +-----+
    | gfdecba |
    +-----+
    ```


## REPEAT

-   Syntax

    ```
    STRING REPEAT(STRING str, BIGINT n)
    ```

-   Description

    This function returns `n` duplicates of the string `str`.

-   Parameters
    -   `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.
    -   `n`: a value of the BIGINT type. The returned string cannot exceed 2 MB in length. An exception is thrown if n is NULL.
-   Return value

    Returns a value of the STRING type.

-   Examples

    ```
    SELECT REPEAT('abc',5) FROM lxw_dual; -- Return abcabcabcabcabc.
    ```


## ASCII

-   Syntax

    ```
    BIGINT ASCII(STRING str)
    ```

-   Description

    This function returns the ASCII code of the first character in the string `str`.

-   Parameters

    `str`: a value of the STRING type. The input value is implicitly converted to a value of the STRING type before computing if it is of the BIGINT, DOUBLE, DECIMAL, or DATETIME type. If the input value is of another data type, an exception is thrown.

-   Return value

    Returns a value of the BIGINT type.

-   Examples

    ```
    SELECT ASCII('abcde') FROM dual; -- Return 97.
    ```


## Additional functions of MaxCompute V2.0

MaxCompute V2.0 extends some mathematical functions. If an additional function that you use in an SQL statement involves new data types, you must insert the following `set` statement before the SQL statement:

```
set odps.sql.type.system.odps2=true;
```

**Note:** Add `set odps.sql.type.system.odps2=true;` before the SQL statement that uses the function, and commit and run it together with the SQL statement so that the new data types can be used.

This section describes new string functions in MaxCompute V2.0.

## CONCAT\_WS

-   Syntax

    ```
    STRING CONCAT_WS(STRING SEP, STRING a, STRING b...)
    STRING CONCAT_WS(STRING SEP, ARRAY)
    ```

-   Description

    This function concatenates all input strings in an array by using a specified delimiter.

-   Parameters
    -   `SEP`: the delimiter of the STRING type. An exception is thrown if you do not specify this parameter.
-   Return value

    Returns a value of the STRING type. NULL is returned if no input parameters are present or an input parameter is NULL.

-   Examples

    ```
    CONCAT_WS(':','name','hanmeimei')='name:hanmeimei'
    CONCAT_WS(':','avg',null,'34')=null
    ```


## LPAD

-   Syntax

    ```
    STRING LPAD(STRING a, INT len, STRING b)
    ```

-   Description

    This function pads the left side of string `a` with string `b` until the new padded string has `len` characters.

-   Parameters
    -   `len`: a value of the INT type.
    -   Parameters such as `a` and `b`: values of the STRING type.
-   Return value

    Returns a value of the STRING type. If `len` is smaller than the number of characters in `a`, `a` is truncated from the left to obtain a string with the number of characters specified by `len`. If `len` is 0, NULL is returned.

-   Examples

    ```
    lpad('abcdefgh',10,'12')='12abcdefgh'
    lpad('abcdefgh',5,'12')='abcde'
    lpad('abcdefgh',0,'12') -- Return NULL.
    ```


MaxCompute V2.0 provides additional date functions. If a function that you use in an SQL statement involves new data types, you must insert the following `set` statement before the SQL statement:

```
set odps.sql.type.system.odps2 = true; -- Enable new data types.
```

## RPAD

-   Syntax

    ```
    STRING RPAD(STRING a, INT len, STRING b)
    ```

-   Description

    This function pads the right side of string `a` with string `b` until the new padded string has `len` characters.

-   Parameters
    -   `len`: a value of the INT type.
    -   Parameters such as `a` and `b`: values of the STRING type.
-   Return value

    Returns a value of the STRING type. If `len` is smaller than the number of characters in `a`, `a` is truncated from the left to obtain a string with the number of characters specified by `len`. If `len` is 0, NULL is returned.

-   Examples

    ```
    rpad('abcdefgh',10,'12')='abcdefgh12'
    rpad('abcdefgh',5,'12')='abcde'
    rpad('abcdefgh',0,'12') -- Return NULL.
    ```


MaxCompute V2.0 provides additional date functions. If a function that you use in an SQL statement involves new data types, you must insert the following `set` statement before the SQL statement:

```
set odps.sql.type.system.odps2 = true; -- Enable new data types.
```

## REPLACE

-   Syntax

    ```
    STRING REPLACE(STRING a, STRING OLD, STRING NEW)
    ```

-   Description

    This function substitutes the string `NEW` for the part of string `a` that is exactly the same as the string `OLD`, and returns the string `a`.

-   Parameters

    The values of all parameters are of the STRING type.

-   Return value

    Returns a value of the STRING type. NULL is returned if an input parameter is NULL.

-   Examples

    ```
    REPLACE('ababab','abab','12')='12ab'
    REPLACE('ababab','cdf','123')='ababab'
    REPLACE('123abab456ab',null,'abab')=null
    ```


## SOUNDEX

-   Syntax

    ```
    STRING SOUNDEX(STRING a)
    ```

-   Description

    This function converts a normal string to a string of the `soundex` type.

-   Parameters

    `a`: a value of the STRING type.

-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    SOUNDEX('hello')='H400'
    ```


## SUBSTRING\_INDEX

-   Syntax

    ```
    STRING SUBSTRING_INDEX(STRING a, STRING sep, INT count)
    ```

-   Description

    This function truncates the string `a` to a substring from the first character to the nth delimiter, where n is specified by `count`. If `count` is a positive value, the string is truncated from left to right. Otherwise, the string is truncated from right to left.

-   Parameters

    `a` and `sep` are of the STRING type, whereas `count` is of the INT type.

-   Return value

    Returns a value of the STRING type. NULL is returned if an input parameter is NULL.

-   Examples

    ```
    SUBSTRING_INDEX('https://www.alibabacloud.com, '.', 2)='https://www.alibabacloud'
    SUBSTRING_INDEX('https://www.alibabacloud.com', '.', -2)='alibabacloud.com'
    SUBSTRING_INDEX('https://www.alibabacloud.com', null, 2)=null
    ```


MaxCompute V2.0 provides additional date functions. If a function that you use in an SQL statement involves new data types, you must insert the following `set` statement before the SQL statement:

```
set odps.sql.type.system.odps2 = true; -- Enable new data types.
```

## URL\_ENCODE

-   Syntax

    ```
    STRING URL_ENCODE(STRING input[, STRING encoding])
    ```

-   Description

    This function encodes the input string in the `application/x-www-form-urlencoded MIME` format and returns the encoded string.

    -   All letters remain unchanged.
    -   Dots \(.\), hyphens \(-\), asterisks \(\*\), and underscores \(\_\) remain unchanged.
    -   Spaces are converted to plus signs \(+\).
    -   Other characters are converted to byte values based on the specified `encoding`. Each byte value is then represented in the format of `%xy`, where `xy` is the hexadecimal representation of the character value.
-   Parameters
    -   `input`: the string to be encoded.
    -   `encoding`: the specified encoding format. GBK and UTF-8 are supported. If you do not specify this parameter, its value defaults to UTF-8.
-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    URL_ENCODE('Example for URL_ENCODE:// dsf(fasfs)', 'GBK') = "Example+for+URL_ENCODE+%3A%2F%2F+dsf%28fasfs%29"
    ```


## URL\_DECODE

-   Syntax

    ```
    STRING URL_DECODE(STRING input[, STRING encoding])
    ```

-   Description

    This function converts an input string from the `application/x-www-form-urlencoded MIME` format to a normal string. This is the inverse function of `URL_ENCODE`.

    -   All letters remain unchanged.
    -   Dots \(.\), hyphens \(-\), asterisks \(\*\), and underscores \(\_\) remain unchanged.
    -   Each plus sign \(+\) is converted to a space.
    -   The `%xy` formatted sequence is converted to byte values. Consecutive byte values are decoded to the corresponding strings based on the input `encoding`.
    -   Other characters remain unchanged.
    -   The final return value of the function is a UTF-8 string.
-   Parameters
    -   `input`: the string to be decoded.
    -   `encoding`: the specified encoding format. GBK and UTF-8 are supported. If you do not specify this parameter, its value defaults to UTF-8.
-   Return value

    Returns a value of the STRING type. NULL is returned if the input parameter is NULL.

-   Examples

    ```
    URL_DECODE('Example+for+URL_DECODE+%3A%2F%2F+dsf%28fasfs%29', 'GBK') = "Example for URL_DECODE:// dsf(fasfs)"
    ```


## JSON\_TUPLE

-   Syntax

    ```
    STRING JSON_TUPLE(STRING json,STRING key1,STRING key2,...)
    ```

-   Description

    This function extracts strings from a standard JSON string``based on specified keys.

-   Parameters
    -   json: a value of the STRING type, which indicates a standard JSON string.
    -   key: a value of the STRING type, which is used to describe the path of a JSON object in the JSON string.``. The value cannot start with a dollar sign \($\). You can enter multiple keys at a time.
-   Return value

    Returns a value of the STRING type.

    **Note:**

    -   NULL is returned if json is empty or invalid.
    -   NULL is returned if key is empty, invalid, or does not exist in the JSON string.
    -   The corresponding string is returned if json is valid and key is also present.
    -   This function can parse JSON data that contains Chinese characters.
    -   This function can parse nested JSON data.
    -   This function can parse JSON data that contains nested arrays.
    -   The parsing action is equivalent to the execution of GET\_JSON\_OBJECT\(\) along with `set odps.sql.udf.getjsonobj.new=true;`. To obtain multiple objects from a JSON string, you must call the GET\_JSON\_OBJECT\(\) function multiple times. As a result, the JSON string is parsed multiple times. The JSON\_TUPLE\(\) function allows you to enter multiple keys at a time and the JSON string is parsed only once. Compared with GET\_JSON\_OBJECT\(\), JSON\_TUPLE\(\) is more efficient.
    -   JSON\_TUPLE\(\) is a User-Defined Table-Generating Function \(UDTF\). If you need to select other columns, use it along with LATERAL VIEW.


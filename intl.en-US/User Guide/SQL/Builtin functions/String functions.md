# String functions {#concept_ulf_pfm_vdb .concept}

This article introduces the string functions such as CHAR\_MATCHCOUNT, CHR, CONCAT, GET\_JSON\_OBJECT, INSTR, IS\_ENCODING supported by MaxCompute.

## CHAR\_MATCHCOUNT {#section_mnd_gvz_vdb .section}

Command format:

``` {#codeblock_7dv_t08_slh}
bigint char_matchcount(string str1, string str2)
```

Usage:

Calculates the total number of times each character in str1 is duplicated in str2.

Parameter description:

-   str1, str2: String type, must be effective UTF-8 strings. If invalid character is in matching process, return a negative value.
-   Return value: Bigint type, Any NULL input, return NULL.

Example:

``` {#codeblock_n7l_k7r_cxl}
char_matchcount('abd','aabc') = 2
-- Two strings 'a', 'b' in str1 appear in str2.
```

## CHR {#section_s5r_lwz_vdb .section}

Command format:

``` {#codeblock_waq_r9g_v8q}
string chr(bigint ascii)
```

Usage:

Convert the specified ASCII code ‘ascii’ into character.

Parameter description:

-   ascii: Bigint type ASCII value. If the input is ‘string’ or ‘double’, it is converted to ‘bigint’ by implicit conversion. If the input is other types, an exception is thrown.
-   Return value: String type. The parameter value range is \[0,255\]. An exception is thrown if exceeding this range. If the input is NULL, return NULL.

## CONCAT {#section_xxj_wwz_vdb .section}

Command format:

``` {#codeblock_tzd_gsb_vaz}
string concat(string a, string b...)
```

Usage:

The return value is a result of connecting all strings.

Parameter description:

-   a, b… String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String by implicit conversion. If the input is other types, an exception is thrown.
-   String: Return value: String type. If no parameter exists or a certain parameter is NULL, return NULL.

Example:

``` {#codeblock_2ad_7dk_iw4}
concat('ab','c') = 'abc'
concat() = NULL
concat('a', null, 'b') = NULL
```

## GET\_JSON\_OBJECT {#section_cdt_gxz_vdb .section}

Command format:

``` {#codeblock_aq6_4my_1wk}
STRING GET_JSON_OBJECT(STRING json,STRING path)
```

Usage:

In a standard json string, the specified string is extracted according to the path.

Parameter description:

-   json: String type, standard json format string.
-   path: **String type, describing the path in json, starting with a dollor sign \($\)**. For a description of the new implementation, see [JsonPath](http://goessner.net/articles/JsonPath/index.html#e2).
    -   $ : Root object
    -   . : Child operator
    -   \[\] : Subscript operator for array
    -   \* : Wildcard for \[\]
-   String: Returns string type.

**Note:** 

-   Return NULL if json is null or invalid json format.
-   Return NULL if path is null or invalid \(does not exist in json\).
-   If json is valid and path also exists, the corresponding string is returned.

Example:

``` {#codeblock_j7k_59i_sso}
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

Use the following query process to extract information in the JSON object:

``` {#codeblock_p6n_1pv_lh7}
odps> SELECT get_json_object(src_json.json, '$.owner') FROM src_json;
amy
odps> SELECT get_json_object(src_json.json, '$.store.fruit\[0]') FROM src_json;
{"weight":8,"type":"apple"} 
odps> SELECT get_json_object(src_json.json, '$.non_exist_key') FROM src_json;
NULL
```

Example:

``` {#codeblock_orq_e5j_6xy}
get_json_object('{"array":[["aaaa",1111],["bbbb",2222],["cccc",3333]]}','$.array[1][1]')= "2222"
get_json_object('{"aaa":"bbb","ccc":{"ddd":"eee","fff":"ggg","hhh":["h0","h1","h2"]},"iii":"jjj"}','$.ccc.hhh[*]') = "["h0","h1","h2"]"
get_json_object('{"aaa":"bbb","ccc":{"ddd":"eee","fff":"ggg","hhh":["h0","h1","h2"]},"iii":"jjj"}','$.ccc.hhh[1]') = "h1"
```

## INSTR {#section_vft_yxz_vdb .section}

Command format:

``` {#codeblock_vea_wkj_5yb}
bigint instr(string str1, string str2[, bigint start_position[, bigint nth_appearance]])
```

Usage:

Calculates where substring str2 is located in str1.

Parameter description:

-   str1: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String by implicit conversion. If the input is other types, an exception is thrown.
-   str2: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String by implicit conversion. If the input is other types, an exception is thrown.
-   start\_position: Bigint type, for other types, an exception is thrown. It indicates from which character of str1 a search must be started from and the default starting position is the first character position 1. If it is less than 0, it causes abnormality.
-   nth\_appearance：bigint type, greater than 0, represents position of the second match of a substring in the string. If the chain is of a different type or less than or equal to 0, an exception is thrown.
-   Return value: Bigint type.

**Note:** 

-   If str2 is not found in str1, return 0.-
-   If any input parameter is null, return null
-   If str2 is NULL and always can be matched successfully, instr \(‘abc’, ‘’\) returns 1.

Example:

``` {#codeblock_3vb_u3l_yus}
instr('Tech on the net', 'e') = 2
instr('Tech on the net', 'e', 1, 1) = 2
instr('Tech on the net', 'e', 1, 2) = 11
instr('Tech on the net', 'e', 1, 3) = 14
```

## IS\_ENCODING {#section_qdj_kyz_vdb .section}

Command format:

``` {#codeblock_ft4_ul3_lh7}
boolean is_encoding(string str, string from_encoding, string to_encoding)
```

Usage:

Determine whether the input string ‘str’ can be changed into a character set ‘to\_encoding’ from a specified character set ‘from\_encoding’. It can be used to Determine whether the input is garbled. The common use is to set ‘from\_encoding’ to be ‘utf-8’ and ‘to\_encoding’ to be ‘gbk’.

Parameter description:

-   str: String type, if the input is NULL, return NULL. The empty string can be assumed to be belonged to any character set.
-   from\_encoding, to\_encoding: String type, source, destination character sets. If the input is NULL, return NULL.
-   Return value: Boolean type. If ‘str’ can be converted successfully, return true, otherwise, return false.

Example:

``` {#codeblock_qbx_l2b_dch}
is_encoding('test', 'utf-8', 'gbk') = true
is_encoding('test', 'utf-8', 'gbk') = true
-- These two traditional Chinese characters are in GBK stock in China.
is_encoding('test', 'utf-8', 'gb2312') = false
-- The grapheme inventory of ‘GB2312’ does not contain these two Chinese characters.
```

## KEYVALUE {#section_lnq_tyz_vdb .section}

Command format:

``` {#codeblock_oz1_nxl_ftq}
KEYVALUE(STRING srcStr,STRING split1,STRING split2, STRING key)
KEYVALUE(STRING srcStr,STRING key) //split1 = ";"，split2 = ":"
```

Usage:

split ‘srcStr’ into ‘key-value’ pairs by split1 and separate ‘key-value’ pairs by split2. Return the value corresponding to key.

Parameter description:

-   srcStr: Source string to be split.
-   key: Specified to return the nth string. After the source string is split by ‘split1’ and ‘split2’, return the corresponding value according to the specification of the ‘key’ value.
-   split1, split2: Strings used as delimiters by which ‘srcStr’ is split. If these two parameters are not specified in the expression, the default value of ‘split1’ is ’;’ and that of ‘split2’ is ’:’. If a string that has been split by split1 and has multiple split2, the return result is not defined.

Return value:

-   String type.
-   If ‘split1’ or ‘split2’ is NULL, return NULL.
-   If ‘scrStr’ and ‘key’ are NULL or in case of no matched ‘key’, return NULL.
-   If multiple ‘key-value’ matches, return the value corresponding to the first matched key.

Example 1:

``` {#codeblock_j0r_16o_7gr}
keyvalue('0:1\;1:2', 1) = '2'
				
```

**Note:** The source string is "0:1\\;1:2". As split1 and split2 are not specified, the default split1 is ";" and split2 is ":".

After the split1 split, the key-value pair is 0:1\\,1:2.

After split2 split, it becomes:

``` {#codeblock_yji_tap_p51}
0 1/  
1 2
```

Returns the value\(2\) of the key corresponding to 1.

Example 2:

``` {#codeblock_83p_6zo_six}
keyvalue("\;decreaseStore:1\;xcard:1\;isB2C:1\;tf:21910\;cart:1\;shipping:2\;pf:0\;market:shoes\;instPayAmount:0\;","\;",":","tf") = "21910" value:21910.
```

**Note:** The source string is as follows:

``` {#codeblock_zdt_594_zdp}
“\;decreaseStore:1\;xcard:1\;isB2C:1\;tf:21910\;cart:1\;shipping:2\;pf:0\;market:shoes\;instPayAmount:0\;”
```

The key-value pairs derived from the split after splitting according to the split1 '\\;' are as follows:

``` {#codeblock_cqd_w8q_vy6}
decreaseStore:1，xcard:1，isB2C:1，tf:21910，cart:1，shipping:2，pf:0，market:shoes，instPayAmount:0 
```

After you split, follow the split2 ":", the results are as follows:

``` {#codeblock_5fs_b3g_dho}
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

The value of the key parameter is "tf", the return value of the corresponding value parameter is 21910.

## LENGTH {#section_ewt_jzz_vdb .section}

Command format:

``` {#codeblock_bz4_lzl_2w7}
bigint length(string str)
```

Usage:

Return the length of a string.

Parameter description:

-   str: String type. If the input is Bigint，Double，Decimal or Datetime, it is converted to String by implicit conversion. If the input is other types, an exception is thrown.
-   Return value: Bigint type. If ‘str’ is NULL, return NULL. If ‘str’ is non UTF-8 coding format, return -1.

Example:

``` {#codeblock_rz5_poo_ov7}
length('hi! China') = 6
```

## LENGTHB {#section_o3y_pzz_vdb .section}

Command format:

``` {#codeblock_ayk_qh1_5uq}
bigint lengthb(string str)
```

Usage:

Return the length of ‘str’ and its unit is byte.

Parameter description:

-   str: String type. If the input is Bigint，Double，Decimal or Datetime, it is converted to String by implicit conversion. If the input is other types, an exception is thrown.
-   Return value: Bigint type. If ‘str’ is NULL, return NULL.

Example:

``` {#codeblock_5tx_n0p_vze}
lengthb('hi!  China') = 10
```

## MD5 {#section_hbw_xzz_vdb .section}

Command format:

``` {#codeblock_s3t_ulp_m47}
string md5(string value)
```

Usage:

Calculate the md5 value of input string.

Parameter description:

-   value: String type. If the input value is of the Bigint, Double, Decimal or Datetime type, it is implicitly converted to the String type before calculation. If the input value is of another type, an exception is thrown. If the input is NULL, return NULL.
-   Return value: String type.

## REGEXP\_EXTRACT {#section_ms1_lc1_wdb .section}

Command format:

``` {#codeblock_rjs_o87_tvf}
string regexp_extract(string source, string pattern[, bigint occurrence])
```

Usage:

Split the string source according to pattern \(regular expression rules\), and return the characters of the occurrence\(nth\) group.

Parameter description:

-   source: String type, a string to be searched.
-   pattern: A string type constant. If pattern is a null string, an exception is thrown. If ‘group’ is not specified in pattern, then also an exception is thrown.
-   Occurrence: A bigint type constant, must be greater than 0 or equal to 0. If it is other type or less than 0, an exception is thrown. If not specified, the default value is 1, which indicates returning the first group. If ‘occurrence’ is equal to 0, then return substrings that satisfy the entire ‘pattern’.
-   Return value: String type. Any input is NULL, return NULL.

Example:

``` {#codeblock_jzz_r2y_v20}
regexp_extract('foothebar', 'foo(. *?)( bar)', 1) = the
regexp_extract('foothebar', 'foo(. *?)( bar)', 2) = bar
regexp_extract('foothebar', 'foo(. *?)( bar)', 0) = foothebar
regexp_extract('8d99d8', '8d(\\d+)d8') = 99
-- If regular SQL is submitted on MaxCompute, two "\" must be used as the shift character.
regexp_extract('foothebar', 'foothebar')
-- The exception is thrown. ‘group’ is not specified in ‘pattern’.
```

## REGEXP\_INSTR {#section_jpn_5c1_wdb .section}

Function definition:

``` {#codeblock_v8h_5rx_e5e}
bigint regexp_instr(string source, string pattern[,
bigint start_position[, bigint nth_occurrence[, bigint return_option]]])
```

Usage:

Returns the start position/end position of the substring, which matches the pattern with the source from start\_position and nth\_occurrence.. Any input parameter is null, return null.

Parameter description:

-   source: String type, to be searched.
-   pattern: A string type constant. If ‘pattern’ is null, an exception is thrown.
-   start\_position: Bigint type constant, the start position of search. If it is not specified, default value is 1. If it is other type or a value is less than or equal to 0, an exception is thrown.
-   nth\_occurrence: A bigint type constant. If not specified, the default value is 1. It appears at the first position, when searched. If it is less than or equal to 0 or other type, an exception is thrown.
-   return\_option: A bigint type constant. Its value is 0 or 1. If it is other type or an invalid value, an exception is thrown. 0 indicates returning the start position of the matched value. 1 indicates returning the end position of the matched value.
-   Return value: Bigint type, the start or end position of a matched substring in source specified by return\_option.

Example:

``` {#codeblock_jto_wns_c6j}
regexp_instr("i love www.taobao.com", "o[[:alpha:]]{1}", 3, 2) = 14
```

## REGEXP\_REPLACE {#section_k2w_2d1_wdb .section}

Command format:

``` {#codeblock_lni_nph_lr9}
string regexp_replace(string source, string pattern, string replace_string[, bigint occurrence])
```

Usage:

replace the substring in source which is matched ‘pattern’ for nth occurrence to be a specified string ‘replace\_string’ and then return.

Parameter description:

-   source: String type, a string to be replaced.
-   pattern: String type constant. The pattern to be matched. If it is null, an exception is thrown.
-   replace\_string: String type, the string after replacing matched pattern.
-   occurrence: Bigint type constant, must be greater than or equal to 0. It indicates replacing nth matching to be replace\_string. If it is 0, it indicates all matched substrings have been replaced. If it is other type or less than 0, an exception is thrown. It can be 0 by default.
-   Return value: String type. When referencing a group which is not existent, do not replace the string. Returns NULL when the source, pattern, occurrence parameter is entered as null, returns NULL, replace\_string is null, but pattern will not match, if the replace\_string is null and the pattern is matched, returns the original string.

**Note:** When the reference group does not exist, it is considered to be undefined.

Example:

``` {#codeblock_mrl_2kr_xwd}
regexp_replace("123.456.7890", "([[:digit:]]{3})\\.([[:digit:]]{3})\\.([[:digit:]]{4})",
"(\\1)\\2-\\3", 0) = "(123)456-7890"
regexp_replace("abcd", "(.)", "\\1 ", 0) = "a b c d "
regexp_replace("abcd", "(.)", "\\1 ", 1) = "a bcd"
regexp_replace("abcd", "(.)", "\\2", 1) = "abcd"
-- Only a group is defined in pattern and the referenced second group is not existent.
-- Please avoid this. The result to reference nonexistent group is not defined.
regexp_replace("abcd", "(. *)(.)$", "\\2", 0) = "d"
regexp_replace("abcd", "a", "\\1", 0) = "bcd"
-- No group definition is in pattern, so '\1' references a nonexistent group,
 -- Please avoid this. The result to reference nonexistent group is not defined.
```

## REGEXP\_SUBSTR {#section_k5b_qd1_wdb .section}

Command format:

``` {#codeblock_f8p_roo_36p}
string regexp_substr(string source, string pattern[, bigint start_position[, bigint nth_occurrence]])
```

Usage:

Starting from start\_position, find a substring in source which matches with a specified pattern for the nth occurrence.

Parameter description:

-   source: String type, string to be searched.
-   pattern: A string type constant. The pattern to be matched. If it is null, an exception is thrown.
-   start\_position: A Bigint type constant, must be greater than 0. Other types or less than equal to 0 throw exceptions. If not specified the default value is 1, which indicates a match begins with the first character of source. If not specified, default value is 1. It indicates a matching value from the first character of source.
-   nth\_occurrence: a Bigint type constant, must be greater than 0. If not specified, the default value is 1. It indicates the return substring of the first matched value. If not specified, the default value is 1. It indicates the return substring of the first matched value.
-   Return value: String type. Any input parameter is NULL, return NULL. If no matching record exists, return NULL.

Example:

``` {#codeblock_vi4_aui_l0g}
regexp_substr ("I love aliyun very much", "a[[:alpha:]]{5}") = "aliyun"
regexp_substr('I have 2 apples and 100 bucks!', '[[:blank:]][[:alnum:]]*', 1, 1) = " have"
regexp_substr('I have 2 apples and 100 bucks!', '[[:blank:]][[:alnum:]]*', 1, 2) = "2"
```

## REGEXP\_COUNT {#section_hxs_yd1_wdb .section}

Command format:

``` {#codeblock_npc_4m3_b5r}
bigint regexp_count(string source, string pattern[, bigint start_position])
```

Usage:

Counts the number of occurrences that a substring matches with a specified pattern, starting from start\_position in source.

Parameter description:

-   Source: String type, the string to be searched. If it is the other type, an exception is thrown.
-   Pattern: String type constant, the pattern to be matched. If it is a null string or other data type, an exception is thrown.
-   start\_position: Bigint type constant, must be greater than 0. If it is other data type or a value which is less than or equal to 0, an exception is thrown. If not specified, default value is 1, which indicates a matched value from the first character of source.
-   Return value: Bigint type. If matching does not exists, return 0. If any input parameter is null, return null.

Example:

``` {#codeblock_f5c_h5j_vzx}
regexp_count('abababc', 'a.c') = 1
regexp_count('abcde', '[[:alpha:]]{2}', 3) = 1
```

## SPLIT\_PART {#section_ecy_k21_wdb .section}

Command format:

``` {#codeblock_u19_v3h_wmo}
string split_part(string str, string separator, bigint start[, bigint end])
```

Usage:

Split the string str according to the separator and return the substring from nth start part to nth end part.

Parameter description:

-   str: String type, the string to be split. If it is Bigint, Double, Decimal or Datetime, it is converted to a String in an implicit conversion. If it is other data type, an exception is thrown.
-   separator: A string type constant, the separator used to split the string. It can be a character or a string. If it is other data type, an exception is thrown.
-   start: A bigint type constant, must be greater than 0. If it is not a constant or other data type, an exception is thrown. It indicates the start number of the return part \(start from 1\). If the end is not specified, returns the part specified by ‘start’.
-   ‘end’: A bigint type constant, must be greater than or equal to ‘start’, otherwise an exception is thrown. It refers to the end number of the return part. If it is not a constant or is other data type, then also an exception is thrown. It can be excluded as it indicates the last part.

Return value: String type. If any parameter is null, return null. If separator is an empty string, return the source string str.

**Note:** 

-   If ‘delimiter’ does not exist in str, then specify ‘start’ as 1, and return the entire str. If the input value is an empty string, the output value is an empty string.
-   If the start value is greater than the number of parts after split, for example, the split produces 6 parts but the ‘start’ value is greater than 6, then returns an empty string.

Example:

``` {#codeblock_top_pq3_3nd}
split_part('a,b,c,d', ',', 1) = 'a'
split_part('a,b,c,d', ',', 1, 2) = 'a,b'
split_part('a,b,c,d', ',', 10) = ''
```

## SUBSTR {#section_nkj_1f1_wdb .section}

Command format:

``` {#codeblock_1fq_18u_xt6}
string substr(string str, bigint start_position[, bigint length])
```

Usage:

Returns a substring of ‘str’ from start\_position with the given length.

Parameter description:

-   ‘str’: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is thrown.
-   The start\_position:Bigint type starts at 1. Returns empty strings when start\_position is 0.When start\_position is negative, the starting position is counted backwards from the end of the string, the last character is -1, and the previous number is -2,-3 and so on. Other types throw exceptions.
-   length: Bigint type, must be greater than 0. If it is other type or less than 0, an exception is thrown. This parameter indicates the length of a child string.
-   Return value: String type. If the input is NULL, return NULL.

**Note:** If the length is excluded, return the substring from start to end.

Example:

``` {#codeblock_6u6_hzk_wzh}
substr("abc", 2) = "bc"
substr("abc", 2, 1) = "b"
substr("abc",-2,2)="bc"
substr("abc",-3)="abc"
```

## SUBSTRING {#section_s1h_3f1_wdb .section}

Command format:

``` {#codeblock_f62_99j_0zh}
string  substring(string|binary str, int start_position[, int length])
```

Usage:

Returns the substring of ‘str’ from start\_position with the given length.

Parameter description:

-   str: String or Binary type, returns NULL or throws an exception for the other type
-   ‘start\_position’: Int type, starting at 1. Returns empty strings when start\_position is 0.When start\_position is negative, the starting position is counted backwards from the end of the string, the last character is-1, and the previous number is in turn-2,-3 and so on. Other types throw exceptions.
-   length: Bigint type, must be greater than 0. If it is other type or less than 0, an exception is thrown. This parameter indicates the length of the child string.
-   Return value: String type. If the input is NULL, return NULL.

**Note:** If the length is excluded, return the substring from start to end.

**For example**:

``` {#codeblock_g4b_jrv_aqn}
substring('abc', 2) = 'bc'
substring('abc', 2, 1) ='"b'
substring('abc',-2,2)='bc'
substring('abc',-3,2)='ab'
substring(BIN(2345),2,3)='001'
```

## TOLOWER {#section_nzz_lg1_wdb .section}

Command format:

``` {#codeblock_1nd_czi_n9h}
string tolower(string source)
```

Usage:

Input the lowercase string corresponding to the English string source.

Parameter description:

-   Source: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   Return Value: String type. If the input is NULL, return NULL.

Example:

``` {#codeblock_bmp_pad_bzw}
tolower("aBcd") = "abcd"
tolower("Haha Cd") = "haha cd"
```

## TOUPPER {#section_qvg_sg1_wdb .section}

Command format:

``` {#codeblock_et1_y5k_1yf}
string toupper(string source)
```

Usage:

Output the uppercase string corresponding to the English string ‘source’.

Parameter description:

-   Source: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   Return Value: String type. If the input is NULL, return NULL.

Example:

``` {#codeblock_iib_det_fqp}
toupper("aBcd") = "ABCD"
toupper("HahaCd") = "HAHACD"
```

## TO\_CHAR {#section_lgq_wg1_wdb .section}

Command format:

``` {#codeblock_rky_cpm_40c}
string to_char(boolean value)
string to_char(bigint value)
string to_char(double value)
string to_char(decimal value)
```

Usage:

Convert Boolean type, Bigint type or Double type to corresponding String type.

Parameter description:

-   Value: Boolean, Bigint or Double type is acceptable. If it is other data type, an exception is thrown. For formatted output of the datetime type, see another function TO\_CHAR that has the same name.
-   Return value: String type. If the input is NULL, return NULL.

Example:

``` {#codeblock_cpk_7zc_5nh}
to_char(123) = '123'
to_char(true) = 'TRUE'
to_char(1.23) = '1.23'
to_char(null) = NULL
```

## TRIM {#section_mf1_3h1_wdb .section}

Command format:

``` {#codeblock_zoa_4ri_91b}
string trim(string str)
```

Usage:

Removes left space and right space for the input string str.

Parameter description:

-   ‘str’: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   Return value: String type. If the input is NULL, return NULL.

## LTRIM {#section_shc_d21_wdb .section}

Command format:

``` {#codeblock_n89_lwn_4kz}
string ltrim(string str)
```

Usage:

Removes the left space for the input string str.

Parameter description:

-   ‘str’: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   Return value: String type. If the input is NULL, return NULL.

Example:

``` {#codeblock_q8y_lj3_5hf}
select ltrim(' abc ') from dual;  
Returns: 
+-----+
| _c0 |
+-----+
| abc |
+-----+
```

## RTRIM {#section_gtk_rh1_wdb .section}

Command format:

``` {#codeblock_okc_604_x8x}
string rtrim(string str)
```

Usage:

Removes the right space for the input string str.

Parameter description:

-   ‘str’: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   Return value: String type. If the input is NULL, return NULL.

Example:

``` {#codeblock_ms7_siz_6hh}
select rtrim('a abc ') from dual; 
Returns:  
+-----+
| _c0 |
+-----+
| a abc |
+-----+
```

## REVERSE {#section_or4_d31_wdb .section}

Command format:

``` {#codeblock_k0d_w14_6uy}
STRING REVERSE(string str)
```

Usage:

Returns a reversed-order string.

Parameter description:

-   str: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   Return value: String type. If the input is NULL, return NULL.

Example:

``` {#codeblock_jjd_om6_e6k}
select reverse('abcedfg') from dual;
Returns: 
+-----+
| _c0 |
+-----+
| gfdecba |
+-----+
```

## SPACE {#section_a25_h31_wdb .section}

Command format:

``` {#codeblock_npr_3yp_vne}
STRING SPACE(bigint n)
```

Usage:

A space string function that returns a string of length n.

Parameter description:

-   n: Bigint type. The length cannot exceed 2 MB. If it is NULL, an exception is thrown.
-   Return value: String type.

Example:

``` {#codeblock_nrz_kfz_1jm}
select length(space(10)) from dual; ----Returns 10.   
select space(400000000000) from dual; ----Error, the length exceeds 2 MB.
```

## REPEAT {#section_gnq_m31_wdb .section}

Command format:

``` {#codeblock_dqg_0of_so1}
STRING REPEAT(string str, bigint n)
```

Usage:

Returns the str string that is repeated for n times.

Parameter description:

-   ‘str’: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   n: Bigint type. The length does not exceed 2 MB. If it is NULL, an exception is thrown.
-   Return value: String type.

Example:

``` {#codeblock_fu4_5g8_tp9}
select repeat('abc',5) from lxw_dual; 
Returns:abcabcabcabcabc
```

## ASCII {#section_hgm_s31_wdb .section}

Command format:

``` {#codeblock_tqm_zkp_peq}
Bigint ASCII(string str)
```

Usage:

Returns the ascii of the first character of str.

Parameter description:

-   str: String type. If the input is Bigint, Double, Decimal or Datetime, it is converted to String in an implicit conversion. If it is other data type, an exception is throwm.
-   Return value: Bigint type.

Example:

``` {#codeblock_e6c_zjw_kw5}
select ascii('abcde') from dual;
Returns:97
```

## Maxcomputerte2.0 Extension function {#section_ncv_srj_m1m .section}

With the upgrade to MaxCompute 2.0, some mathematical functions have been added to the product. If a new function uses a new data type, you must add the following set statement before using the new functions SQL statement:

``` {#codeblock_ibu_a4w_eai}
set odps.sql.type.system.odps2=true;
```

**Note:** Add set odps before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true;`, and commit runs with SQL to use the new data type function normally.

The enhanced and extended string functions are described as follows.

## CONCAT\_WS {#section_xnf_sj1_wdb .section}

Command format:

``` {#codeblock_f0i_1s2_3rt}
string concat_ws(string SEP, string a, string b...)
string concat_ws(string SEP, array)
```

**Note:** Add set odps before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true;`, and commit runs with SQL to use the new data type function normally.

Usage:

Concatenates all strings in the parameters, connected by the specified delimiter.

Parameter description:

-   SEP: String-type delimiter. If not specified, an exception is returned.

Return value:

String type. If no parameters exist or any parameter is null, return null.

Example:

``` {#codeblock_834_9be_2fy}
concat_ws(':','name','hanmeimei')='name:hanmeimei'
concat_ws(':','avg',null,'34')=null
```

## LPAD {#section_mcj_zj1_wdb .section}

Command format:

``` {#codeblock_6by_hie_39l}
string lpad(string a, int len, string b)
```

**Note:** Add set odps before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true;`, and commit runs with SQL to use the new data type function normally.

Usage:

Uses string b to pad string a to the left to the place specified by len.

Parameter description:

-   len: Int-type integer.
-   a/b…: String type.

Return value:

String type. If len is smaller than the number of places in a, a is truncated from the left to obtain a string with the number of places specified by len. If len is 0, return empty.

Example:

``` {#codeblock_whw_cdg_r19}
lpad('abcdefgh',10,'12')='12abcdefgh'
lpad('abcdefgh',5,'12')='abcde'
lpad('abcdefgh',0,'12') Returns a blank result
```

## RPAD {#section_k1f_3k1_wdb .section}

Command format:

``` {#codeblock_zsc_mz4_ews}
string rpad(string a, int len, string b)
```

**Note:** Add set odps before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true;`, and commit runs with SQL to use the new data type function normally.

Usage:

Uses string b to pad string a to the right to the place specified in len.

**Note:** 

You need to add the set odps statement before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true`, otherwise the error is reported.

Parameter description:

-   len: Int-type integer.
-   a/b…: String type.

Return value:

String type. If len is smaller than the number of places in a, a is truncated from the left to obtain a string with the number of places specified by len. If len is 0, return empty.

Example:

``` {#codeblock_gyh_9ca_tq6}
rpad('abcdefgh',10,'12')='abcdefgh12'
rpad('abcdefgh',5,'12')='abcde'
rpad('abcdefgh',0,'12') Returns a blank result
```

## REPLACE {#section_ln3_5k1_wdb .section}

Command format:

``` {#codeblock_601_oew_430}
string replace(string a, string OLD, string NEW)
```

**Note:** Add set odps before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true;`, and commit runs with SQL to use the new data type function normally.

Usage:

Uses string NEW to replace the portion of string a that completely matches string OLD and returns string a.

Parameter description:

The parameters are all String type.

Return value:

String type. If the input is null, return null.

Example:

``` {#codeblock_o8h_wy1_10d}
replace('ababab','abab','12')='12ab'
replace('ababab','cdf','123')='ababab'
replace('123abab456ab',null,'abab')=null
```

## SOUNDEX {#section_z2z_1l1_wdb .section}

Command format:

``` {#codeblock_n0a_4v5_uh6}
string soundex(string a)
```

**Note:** Add set odps before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true;`, and commit runs with SQL to use the new data type function normally.

Usage:

Converts a normal string to a soundex string.

Parameter description: a is of type String.

Return value: String type. If the input value is NULL, return NULL.

Example:

``` {#codeblock_qgc_3fb_pfl}
soundex('hello')='H400'
```

## SUBSTRING\_INDEX {#section_uw3_hl1_wdb .section}

Command format:

``` {#codeblock_mm0_hij_l0x}
string substring_index(string a, string SEP, int count))
```

**Note:** Add set odps before the SQL statement that uses the function`set odps.sql.type.system.odps2 = true;` and commit runs with SQL to use the new data type function normally.

Usage:

Truncates string a to the portion in front of the delimiter specified by count. If count is positive, the portion to the left of the delimiter is used. If count is negative, the portion to the right is used.

Parameter description: a/sep belong to the string type, and count belongs to the int type.

Return value:

String type. If the input is null, return null.

Example:-

``` {#codeblock_n5r_rkr_fil}
substring_index('https://help.aliyun.com', '.', 2)='https://help.aliyun'
substring_index('https://help.aliyun.com', '.', -2)='aliyun.com'
substring_index('https://help.aliyun.com', null, 2)=null
```


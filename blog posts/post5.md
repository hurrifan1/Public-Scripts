# Encoding strings in Base64 using Qlik script

I recently had a project requirement that involved needing to encode a few strings in Base64 to be sent over a REST API call. As happens often during projects, I was able to find a different solution that precluded the need for lots of Qlik script...but I had already started, so I just went ahead and finished it!

Below, I go through the process of writing some Qlik script to encode strings in Base64.

## Step 1. What the hell is Base64?
Yeah, I didn't know before this project. Is it encryption? Is it a way to convert between character encodings?

Turns out, it's _basically_ a way to get from binary data to textual data, and vice versa, for the purpose of transferring information. That's no doubt a horrendous over-simplification, perhaps to the point of being a bit wrong, but it's near enough for the purposes of this post.

## Step 2. Determine the game plan
Here's what I needed my script to do:

1. Take any string (i.e. "Qlik").
2. Convert each character to their ASCII index.
3. Convert those ASCII index values to ~~binary~~ _(turns out, "bit pattern" is the more apt term here)_. Note that these are octets, or groups of 8 bits.

Here's a table to illustrate what we have so far:

|     Q    |     l    |     i    |     k    |
|:--------:|:--------:|:--------:|:--------:|
|    81    |    108   |    105   |    107   |
| 01010001 | 01101100 | 01101001 | 01101011 |

4. Combine those bits and then separate them into 24-bit sequences. 

|          24 bits         |  ~~24 bits~~ 8 bits |
|:------------------------:|:--------:|
| 010100010110110001101001 | 01101011 |

Notice that with our string "Qlik", we end up with a group of bits that don't fit into a 24-bit sequence -- in this case, we have 8 leftover. What we'll do is add zeroes to the end of that octet, which will ultimately be converted to a padding character, = (equal sign):

|          24 bits         |          24 bits         |
|:------------------------:|:------------------------:|
| 010100010110110001101001 | 011010110000000000000000 |

5. Then, split those groups into sextets, or groups of six.
6. Those sextets are then converted to an index that will return a Base64 character.

When using our above example, the first 6 bits in our 24-bit sequence are 010100. This converts to the number 20. So we now must go to that index in this string of Base64 characters:
`ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/`

Doing so will return the character U:

```
0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 .....
A  B  C  D  E  F  G  H  I  J  K  L  M  N  O  P  Q  R  S  T  U  V  W  X  Y  Z  a  b  c  d  e  f  .....
                                                            ^
                                                        Index 20
                                                     (0-based index)
```

Here's where we are in our example:

| 010100 | 010110 | 110001 | 101001 | 011010 | 110000 | 000000 | 000000 |
|:------:|:------:|:------:|:------:|:------:|:------:|:------:|:------:|
|   20   |   22   |   49   |   41   |   26   |   48   |   pad  |   pad  |
|    U   |    W   |    x   |    p   |    a   |    w   |    =   |    =   |

Note that a bit pattern of 000000 would normally be the index for the chartacter A but in this case, since we added -- or padded -- the end with these zeroes, they're converted to the padding character, =.

## Step 3. Script it out in Qlik
Now time for the tricky part.

The basic thing I'll aim for is this:
- Write a subroutine that will:
  - Take a string argument;
  - Base64 encode the string;
  - Set the encoded string to a variable.

Let's start with this script here:

```
SUB Base64Encode (string)

    Let vbase_String = '$(string)';
    Let vbase_Base64Chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
    Let vbase_QuotePad = 1;

    Trace #### Our string = $(vbase_String);

END SUB
```

This sets up our subroutine along with a few variables:
- `vbase_String` - Takes the subroutine argument value.
- `vbase_Base64Chars` - Holds the Base64 characters.
- `vbase_QuotePad` - This variable is basically here to allow for future functionality; its current purpose is to allow for double-quotes around the provided string so that Qlik can read tricky characters like quotes " " and semicolons ;.

We also have a TRACE in there just to confirm in the script output that our string was picked up correctly.

Now we'll generate a table where we have:
- A field for each character in our string;
- A field for keeping track of the order of the characters;
- and a field for tracking the byte in base 2 (binary!).

Here's the script for that:

```
SUB Base64Encode (string)
...

base_string_table_a:
    LOAD *
      , RowNo() as base_Character_Number
      , text(repeat(0, 8 - len(num(ord(base_Character), '(BIN)'))) & num(ord(base_Character), '(BIN)')) as base_Binary
    ;
    LOAD
        Mid( mid('$(vbase_String)', 1+$(vbase_QuotePad), len('$(vbase_String)')-($(vbase_QuotePad)*2)),  IterNo(),  1 ) as base_Character
    AutoGenerate 1 While IterNo() <= len('$(vbase_String)')-($(vbase_QuotePad)*2);
    
END SUB
```

Above, we used AutoGenerate with IterNo() and Mid() to parse each character into a separate row (field `[base_Binary]`). Then we have a preceding load that keeps track of the character order (field `[base_Character_Number]`) and converts the ASCII indices of the characters to binary octets, making sure to add zeroes to the beginning to ensure 8 bits, as Qlik will otherwise strip those zeroes (field `[base_Binary]`).



```
SUB Base64Encode (string)
...
...
    
    base_full_binary:
    LOAD *
/* 6 */  , RowNo() as base_Binary_Index
/* 5 */  , Mid('$(vbase_Base64Chars)', base_Base64_Index + 1, 1) as base_Base64_Character
    ;
    LOAD *
/* 4 */  , num(num#(base_New_Bit_Pattern, '(BIN)')) as base_Base64_Index
    ;
    LOAD *
/* 3 */  , mid(base_Full_Binary, ( ( IterNo() - 1 ) * 6 ) + 1, 6) as base_New_Bit_Pattern
    WHILE IterNo() <= Div( len(base_Full_Binary) , 6)
    ;
    LOAD *
/* 2 */  , text( base_Full_Binary_No_Pad & Repeat(0, num#(Frac(Fabs((Mod(len(base_Full_Binary_No_Pad), 24) - 24) / 6)) * 6) ) ) as base_Full_Binary
    ;
    LOAD
/* 1 */  text( Concat(base_Binary, '', base_Character_Number) ) as base_Full_Binary_No_Pad
    Resident base_string_table_a;

END SUB
```

So there's a lot going on here. I've added some numbers `/* # */` in the script to help you follow along:
1. This loads a new table by residenting the table created in the script section we discussed above. This line concatenates all of the binary octets from the previous section (results in field `[base_Full_Binary_No_Pad]`).
2. This preceding load adds the field `[base_Full_Binary]`, which adds enough zeroes to the end of our concatenated binary string to complete the last sextet, which we'll be creating in the next step (we'll add the padding in a later step).
3. This script adds a new field, `[base_New_Bit_Pattern]`, that breaks out the binary string into sextets, or groups of 6.
4. This line converts those sextets into a number (resulting in field `[base_Base64_Index]`).
5. This line uses the `[base_Base64_Index]` field as an index to pull the Base64 character from the `vbase_Base64Chars` variable we created at the start (results in field `[base_Base64_Character]`).
6. Finally, this line adds the field `[base_Binary_Index]`, which provides a row number the we can use to keep these characters in order.

Now we're ready to put our Base64 encoded string together and add the padding character, if needed:

```
SUB Base64Encode (string)
...
...
...
    
    let vbase_Len1 = len(peek('base_Full_Binary', 0, 'base_full_binary'));
    let vbase_Len2 = len(peek('base_Full_Binary_No_Pad', 0, 'base_full_binary'));


    base_Base64_Encoded:
    LOAD
        Concat(base_Base64_Character, '', base_Binary_Index) & repeat('=', div( 24 - mod($(vbase_Len2), 25) - num#(Frac(Fabs((Mod($(vbase_Len2), 24) - 24) / 6)) * 6) , 6) ) as base_Base64_Encoded_String
    Resident base_full_binary;


    Let vBase64String = peek('base_Base64_Encoded_String', 0, 'base_Base64_Encoded');

END SUB
```

We first created two new variables:
- `vbase_Len1` - The length of the binary string after we added any needed extra zeroes to complete the last sextet.
- `vbase_Len2` - The length of the binary string before we had done that step mentioned above.

Then, we resident the previous table and create a new field, `[base_Base64_Encoded_String]`, by concatenating our Base64 encoded characters from the field `[base_Base64_Character]` and then adding the padding characters we need so that we keep with our 24-bit sequence length.

We now have our Base64 encoded string! We create a variable, `vBase64String`, to hold that string.

The final step is simply some housekeeping:

```
SUB Base64Encode (string)
...
...
...
...

    Trace #### Encoded string available from the vBase64String variable!;

    Drop Tables [base_Base64_Encoded], [base_full_binary], [base_string_table_a];
    Let vbase_String = Null();
    Let vbase_Base64Chars = Null();
    Let vbase_Len1 = Null();
    Let vbase_Len2 = Null();

END SUB

```

This script simply adds a TRACE to confirm that this all worked and then drops our tables and nullifies our helper variables.

You can now use this subroutine as such:

```
Let string = '"' & 'Place your string here!' & '"';

Call Base64Encode ('$(string)')
```

You'll notice that you have to wrap your string with double-quotes -- this is to ensure that Qlik can parse out Qlik-specific characters, like semicolons and quotes/double-quotes.

## Final thoughts

And there ya go! This script will encoded a string in Base64. It was hard to figure out, as I didn't anticipate how much math would go into it. Someone who knows binary math and/or Qlik script better than I do can probably find a ton of ways to make this more efficient but certainly gets the job done!

### Notes
I used several excellent resources to help me muscle through this one. Those are linked below:
- [base64encode.org](https://www.base64encode.org/)
- [Good ol' Wikipedia](https://en.wikipedia.org/wiki/Base64)

## Full script

```
SUB Base64Encode (string)

    Let vbase_String = '$(string)';
    Let vbase_Base64Chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
    Let vbase_QuotePad = 1;

    Trace #### Our string = $(vbase_String);
 

    // ####################################################################


    base_string_table_a:
    LOAD *
      , RowNo() as base_Character_Number
      , text(repeat(0, 8 - len(num(ord(base_Character), '(BIN)'))) & num(ord(base_Character), '(BIN)')) as base_Binary
    ;
    LOAD
        Mid( mid('$(vbase_String)', 1+$(vbase_QuotePad), len('$(vbase_String)')-($(vbase_QuotePad)*2)),  IterNo(),  1 ) as base_Character
    AutoGenerate 1 While IterNo() <= len('$(vbase_String)')-($(vbase_QuotePad)*2);


    base_full_binary:
    LOAD *
      , RowNo() as base_Binary_Index
      , Mid('$(vbase_Base64Chars)', base_Base64_Index + 1, 1) as base_Base64_Character
    ;
    LOAD *
      , num(num#(base_New_Bit_Pattern, '(BIN)')) as base_Base64_Index
    ;
    LOAD *
      , mid(base_Full_Binary, ( ( IterNo() - 1 ) * 6 ) + 1, 6) as base_New_Bit_Pattern
    WHILE IterNo() <= Div( len(base_Full_Binary) , 6)
    ;
    LOAD *
      , text( base_Full_Binary_No_Pad & Repeat(0, num#(Frac(Fabs((Mod(len(base_Full_Binary_No_Pad), 24) - 24) / 6)) * 6) ) ) as base_Full_Binary
    ;
    LOAD
        text( Concat(base_Binary, '', base_Character_Number) ) as base_Full_Binary_No_Pad
    Resident base_string_table_a;


    let vbase_Len1 = len(peek('base_Full_Binary', 0, 'base_full_binary'));
    let vbase_Len2 = len(peek('base_Full_Binary_No_Pad', 0, 'base_full_binary'));


    base_Base64_Encoded:
    LOAD
        Concat(base_Base64_Character, '', base_Binary_Index) & repeat('=', div( 24 - mod($(vbase_Len2), 25) - num#(Frac(Fabs((Mod($(vbase_Len2), 24) - 24) / 6)) * 6) , 6) ) as base_Base64_Encoded_String
    Resident base_full_binary;


    Let vBase64String = peek('base_Base64_Encoded_String', 0, 'base_Base64_Encoded');
    Trace #### Encoded string available from the vBase64String variable!;

    Drop Tables [base_Base64_Encoded], [base_full_binary], [base_string_table_a];
    Let vbase_String = Null();
    Let vbase_Base64Chars = Null();
    Let vbase_Len1 = Null();
    Let vbase_Len2 = Null();

END SUB
```

# PCBID V2.0 specifications - 3nd Draft

![Akornsys RDI](https://github.com/akornsys-rdi/pcbid-specifications/raw/master/doc/img/akornsys-logo.png)

## :rocket: Description and purpose

PCBID is a unique numbering system for PCBs designed to generate **unique codes** that identify PCBs and allow their **traceability**. This numbering has been conceived to be a robust system, **usable in any scenario**. It also **provides useful information** for the user, ensuring a **good readability** of the numbering.

The system has been designed to provide a **high data compression**, creating a **code short enough** to be inserted in even the smallest PCB. And the generation process, as well as the overlay process, allows to be **automated processes** with external tools. In this way, close to 30 million authors are allowed in the current version, with a total of almost 900 trillion unique projects.

### Recommended silkscreen size

Just as there is no single way to make a PCB, there is no single size that is considered good for printing PCBID on a PCB. However, we consider that in order to maintain good readability, it should not be less than 30x6mil.

### Character set

PCBID uses K74 encoding, an ASCII85-inspired charset that replaces or removes those characters that can be confused with others, to avoid typographical confusion.

:warning: Charset modified in the third draft of PCBID V2.0. This breaks the compatibility with previous versions and the external tools that handled the previous version.

K74 consists on the following charset:

        1                 10                  20                  30
        
        0 1 2 3 4 5 6 7 8 9 A B C E F G H J K L M N P Q R S T U V W
        X Y Z a b c d e f g h j k l m n p q r s t u v w x y z ! # $
        % & ( ) + - ; < = > ? _ [ ]

This charset has been carefully considered. It is thus excluded:
- `"I"`, `"|"` and `"i"`, which can be confused with , `"1"`, `"j"` `"l"`.
- `"O"`, `"o"` and `"D"`, which can be confused with `"0"` and `"Q"`.
- `"{"`, which can be confused with `"["` and `"("`.
- `"}"`, which can be confused with `"]"` and `")"`.
- `":"`, which can be confused with `";"`.
- `"\"` and `"/"`, which combined can be confused with `"V"`. This also allows URI encoding and not to be confused with a escape character.
- `"@"`, for looking like a blob in some type fonts.
- ``"`"``, `"'"`, `'"'`, `"*"` and `"^"` which are too small and may not be legible in silkscreen.
- `" "`, for being a non-printable character.
- `"~"`, which can be confused with `"-"`. No longer used as a prefix, see Use of tilde prefix section for further information.

### Description of fields

The PCBID code consists of six fields:
- **Author**: Identifier of the person or organization developing the PCB.
- **Project**: Identifier of the project developed by the author.
- **Module**: Numbering of the board that belongs to the project. Oriented for projects with more than one PCB design. The field starts with 01.
- **Release**: Numbering of the release of this module. Any type of release increases this field. The field starts with 01.
- **Week**: Week in which the PCB layout ends (ISO 8601 model).
- **Year**: Year in which the PCB layout ends.

The author and project identifiers are generated from the actual names of these. An algorithm transforms them into a compact and unique identifier to form the PCBID fields.

:warning: By style convention intended for the readability and accuracy of the data provided, the author name and project name fields must present full names, without unnecessary abbreviations and with the correct capitalization and punctuation.

The canonical structure of the fields is designed in the form of dependency, so the second field and successive fields are dependent on the previous field. A change in one field forces the resetting of the successive fields. This way the fields can be read in the opposite order: _Design date_ OF _the release_ OF _the module_ OF _the project_ OF _the author_.

This is how PCBID is composed:

         aaaappppmmrrwwyy        ]S3y1%kZ01011520
         aaaa                    ]S3y               Author      Foobar Incorporation
             pppp                    1%kZ           Project     Foobar Project
                 mm                      01         Module      1
                   rr                      01       Release     1
                     ww                      15     Week        15
                       yy                      20   Year        2020

Separated by identifiers, the _Author identifier_ (**Author ID**) is composed of the **Author** field, while the _Project identifier_ (**Project ID**) is composed of the **Author** and **Project** fields:

         aaaappppmmrrwwyy        ]S3y1%kZ01011520   PCBID
         aaaa                    ]S3y               Author ID
         aaaapppp                ]S3y1%kZ           Project ID

### 2D Barcode

The numbering of PCBIDs can also be encoded in barcode format to be added in the silkscreen of the PCB. The barcode type for PCBID coding is **datamatrix**. In all cases, the use of a 2D barcode is strongly recommended, as it offers easy location on the PCB, better readability from external devices, possibility of process automation and error correction in a compact size. No other information should be encoded in the 2D barcode. It is highly recommended to add the PCBID code in plain text in a location close to the barcode whenever possible.

It works well to generate it in inverted colors and with a size in PCB not less than 6mm, as long as the color of the mask offers enough contrast. Remember to keep a safety contour around the 2D barcode. The generation of the 2D barcode can be done in an automatic way from external tools.

### Use of tilde prefix

:warning: The use of the tilde has changed in the third draft of PCBID V2.0. This breaks the compatibility with previous versions and the external tools that handled the previous version.

The tilde symbol `~` was used as a prefix in PCBID to indicate that a text string is a PCBID number and thus distinguish it from arbitrary looking text strings. Its use is now deprecated.

### When it should be generated?

In order for the date embedded in the PCBID code to be valid, it must be generated and inserted once the PCB is finished, at the moment prior to generating the manufacturing files. This code will be valid and should not be updated until the next change in the PCB.

### How it is generated?

Tools for PCBID generation are provided in the [PCBID Tools](https://github.com/akornsys-rdi/pcbid-tools) repository. If you are interested in creating your own generation tool or simply to know more details, you can look at the technical details section.

The [PCBIDB](https://pcbidb.akornsys-rdi.net) parallel project will offer, when completed, online generation and an API for generation with external tools.

### Examples of use

#### Case 1: First PCB

1. Generate your author identifier from the author name you want (personal or brand name). E.g. `]S3y` for `Foobar Incorporation`.
2. Generate your project identifier from the project name. E.g. `1%kZ` for `Foobar Project`.
3. Set the module and release fields to `01`.
4. Calculate the current week following the ISO 8601 model (`%V` in `date` syntax). E.g. `15` for `2020-04-11`.
5. Add the last two digits of the year for the year field. E.g: `20` for `2020`.
6. Concatenate the fields, this way it has to look like this: `]S3y1%kZ01011520`.
7. Insert the PCBID on the PCB.

#### Case 2: Second version of the PCB

1. Take the PCBID that was assigned to the previous version of this PCB. E.g: `]S3y1%kZ01011520`.
2. Increase by one unit, following the K74 charset, the release field. E.g: `]S3y1%kZ01021520`.
3. Recalculate the date when you have completed the changes to the second version of the PCB. E.g: `]S3y1%kZ01021820`.
4. Insert the PCBID on the PCB.

#### Case 3: New board in the same project

1. Take the author and project fields you had generated from this project. E.g: `]S3y1%kZ`.
2. Increase by one unit, following the K74 charset, the module field. E.g: `02`.
3. Restart the release field, as this is the first release of this PCB. E.g: `01`.
4. When you have finished the design, calculate the date fields. E.g: `2120`.
5. Concatenate the fields, this way it has to look like this: `]S3y1%kZ02012120`.
6. Insert the PCBID on the PCB.

### Conclusion

In this way, the author Foobar Incorporation (`]S3y`) has a single project (`1%kZ`), which contains two PCBs:
- The first PCB in its second version, with the PCBID `]S3y1%kZ01021820`.
- The second PCB in its first version, with the PCBID `]S3y1%kZ02012120`.

### FAQ

> **Some time ago I designed and manufactured some PCBs with PCBID, now I want to manufacture more PCBs from the same manufacturing files. Should I update the PCBID?**
> 
> In the regular use of PCBIDs it is not necessary to update the numbering. However, if it is an event that you need to record for traceability, it is legitimate to increase the release field by one unit each time new PCBs are to be manufactured.

> **I have designed a project of several boards and all of them have PCBID. I have created a second version of one of them, do I have to update the PCBID in all of them?**
> 
> No, you only need to update the release field by increasing a unit on the board from which you have created a second version.

> **Should I update the PCBID if I had designed a PCB and after some time I make changes, without having manufactured any unit of this PCB?**
> 
> Yes. Make all the necessary changes and updates only the date fields if necessary.

> **Can I use this system for the non-PCB housing or hardware of my project?**
> 
> It is not the intention of the PCBID and it is probably not the most convenient option. PCB manufacturing processes are far away from other hardware manufacturing processes, and this hardware may need to be traced in a different way than with PCBs. In any case you are free to use it if you wish.

> **How it can have traceability if all the PCBs manufactured from the same manufacturing files have the same PCBID?**
> 
> PCBID is based on the idea that all boards manufactured from the same manufacturing files are virtually identical, so all traceability events affect them equally. It therefore focuses more on the changes a PCB undergoes from the early proto to the final version than on those individual events. However, we know that there are situations in which it is necessary to keep track of traceability on an individual basis, in which case we recommend adding another numbering system in parallel, such as numbered stickers.

> **How does the system avoid possible collisions of author or project identifiers?**
> 
> Collision checking is implemented in the generation tools. In the case of offline tools, it is done through a common file that stores the **Project ID** generated at the time by any author to check for collisions. The online tools use a common database that checks for collisions. In any case, collisions are very unlikely and it is reasonably safe to generate these fields without any checks for projects by the same author.

> **What is PCBIDB?**
> 
> PCBIDB is a parallel project that is under development and will allow the online generation of PCBIDs and store in a database all the PCBIDs created, linking all the traceability events, change logs, documentation and other information related to each board with a PCBID.

> **Can I use this system for my software projects?**
> 
> It is not the intention of the PCBID and it is probably not the most convenient option. Software version control systems bring their own advantages and include their own traceability systems. In any case you are free to use it if you wish.

> **My project has changed name, should I change the project field of the PCBID?**
> 
> If you have not yet manufactured any PCBs from this project with the old name, generate the PCBID with the new project name. However, if you have already manufactured PCBs of the project with the old name and it has a PCBID assigned, you should keep the project identifier of the PCBID you have already generated for the boards with the new project name, for traceability reasons. If the reason for the project name change is a structural change of the project, maybe you should treat it as a new project and generate your own project identifier. Ultimately it's up to you.

> **Why this numbering system and not any other?**
> 
> In the world of PCB manufacturing there are numerous numbering systems with the intention of having traceability, but none of them are standard and their scope is usually for internal use by each company or organization. Therefore, PCBID has been designed to create standard specifications that can be adopted by anyone who might be interested. In addition, this allows the creation of a database of all those using the same system (PCBIDB). Of course you are free to use the system that suits you best.

> **I am designing a project ecosystem consisting of several PCBs, should I change the project field or the module field for each of them?**
> 
> It depends. It's a semantic issue, but if you can understand that each board is a project in itself and doesn't require others from its ecosystem to function you should probably change the project field. For example different cards of a rack synthesizer, the grouping of them forms a unit but each one is autonomous separately. If, on the other hand, there is a certain dependency between the different boards in order to work, you probably have to change the module field. For example a main board with an set of expansion boards. Ultimately it's up to you.

> **I'm going to make a copy of a PCB designed by someone else and containing PCBID, should I keep it?**
> 
> In general it is a good practice to keep it, as this way the traceability history of the original author can be accessed and the authorship of the project recognized, which is always good. If you don't want this, you can delete it or regenerate it with your own author identifier, keep the project field and restart the release field.

> **I designed a very small PCB and I can't fit the PCBID, can I shorten it?**
> 
> No. All the fields that form PCBIDs are relevant and some cannot be ruled out. You can reduce the font size to the recommended minimum or if you have already done so you can consider other options, such as inserting it in two lines, or perhaps inserting it in datamatrix format. If you choose to put it on two lines, you should preferably do so in the following format:
> 
>       aaaapppp            ]S3y1%kZ
>       mmrrwwyy            01011520

> **Can I choose the author and/or project identifier using K74 alphabet?**
> 
> No, these fields cannot be chosen arbitrarily and must be generated from the author and project name. This process ensures unique identifiers linked to the source name and prevents collisions.

## :wrench: Technical details

The PCBID generation algorithm depends on the field being generated:
- Author and project: The author and project identifier fields are generated from the hash of the name provided. This function guarantees a correspondence between the name and the identifier, while minimizing collisions. The hash is processed to obtain a radix 74 encodable number.
- Release and Module: These fields are the representation of the decimal number in radix 74. Since these fields will rarely exceed 09, they are kept clearly human readable.
- Week and year: Two-digit decimal representation of the week and year. Without any coding, they are easy to read.

All numbers in radix 74 are represented with the K74 charset, which provides a comfortable reading without possible confusion by similar characters.

The hash function chosen for the author and project name transformation is SHA256, which provides a solid hash feature. The hash transformation algorithm is performed by breaking the hash result into 32-bit chunks, and adding these chunks together. This process has proven to have better data dispersion.

        SHA256("Foobar Incorporation")
                |
                v
        49d25a4f106d4dacca67b2d3e131d8e1bfbf942edf68997f1b473c4b4fee89aa
        49d25a4f
              + 106d4dac
                      + ca67b2d3
                              + e131d8e1
                                      + bfbf942e
                                              + df68997f
                                                      + 1b473c4b
                                                              + 4fee89aa
        ________________________________________________________________
                                                              0410372751

The result is divided by the maximum possible value of the identifier (74⁴), and the remainder is converted to radix 74.

             RADIX 10        RADIX 16        RADIX 74
            17451919185     0410372751      07 63 73 25 03 55
        MOD    29986576       01c98f10         01 00 00 00 00
        _____________________________________________________
               29718529       01c57801            73 25 03 55

The number in radix 74 is represented using the K74 charset, forming the identifier.

        73 25 03 55  ->  ]S3y

The same process applies to the project name.

In case of finding collisions with existing PCBID identifiers, `" #<NUMBER>"` is added at the end of the author or project name which causes the collision, where `<NUMBER>` will increase by one unit for each collision found. This appended is not part of the author or project name, and should not be added to collision tracking systems, such as the `projects.txt` file (used by external offline generation tools).

For the author, collisions are searched only in the **Author ID** (first four characters of the PCBID), while for the project, collisions of the same author are searched (**Project ID**, first eight characters of the PCBID). Therefore, the same project identifier is allowed for different authors.

An example of collisions would be, the (new) author _Foobar Incorporation_ tries to generate the project identifier of _Foobar Project_. According to the collision tracking system, the author identifiers `]S3y` and `2)z>` are already in use:

        "Foobar Incorporation"     -> 49d25a4f..4fee89aa -> ]S3y      --> COLLISION!
        "Foobar Incorporation #1"  -> 0b311707..a5440351 -> 2)z>      --> COLLISION!
        "Foobar Incorporation #2"  -> c09b75f3..9842a017 -> <W[g      --> ASIGNED
                                                              |
                                                              v
        "Foobar Project" -> 78160470..7b1566af ->           ____ 1%kZ --> ASIGNED

Thus, the assigned PCBID identifiers would be `<W[g1%kZ`, since the first collision-free author identifier and the first collision-free project identifier for that author are used. Once an author has an assigned identifier, they must use it for all their projects.

## :book: Version history

### PCBID V2.0 3rd Draft

The changes from the 2nd draft are:
- Updated title to 3rd Draft.
- BREAKING: changed the charset and added explanations of the ignored characters on the charset.
- Updated the number of authors and projects possible with the new charset.
- Added explanatory notes in Description of fields and modified paragraphs that were confusing.
- Added description of Author ID and Project ID identifiers.
- Updated Foobar Inc to Foobar Incorporation according to style convention.
- Now it is recommended to use 2D barcode versus plain text.
- Added usage notes on the 2D barcode.
- BREAKING: Removed the use of the tilde. Added section on the Use of tilde prefix.
- Added paragraph in the section How it is generated using the PCBIDB web.
- Updated Examples of use to comply with the rest of the changes.
- Rewritten the conclusion paragraph of the Examples of use.
- Clarified FAQ answers.
- Updated Technical details with the new charset.

### PCBID V2.0 2nd Draft

The changes from the 1st draft are:
- Updated title to 2nd Draft
- Edited FAQ
- Updated sample PCBID
- Added paragraph about the formation of the author name and project name fields
- Added section for technical details
- Added how to generate it
- Modified barcode and pull-request sections
- Simplified file system to a single file
- Updated content of projects.txt
- Added technical details about collisions

### PCBID V1.0

The previous version of PCBID is considered obsolete, and is retained only as documentation. This version was designed to be used by a single author as a way to generate serial numbers for personal projects

        SN:k7759C62C01a_4618
        SN:                     V1 mark
           k                    Author
            7759C62C            CRC32 of the project URI in HEX
                    01          PCB module number
                      a         PCB release
                       _        Spacer
                        46      Week
                          18    Year

## :heart: Community and contributions

[![Donate using Liberapay](https://github.com/akornsys-rdi/pcbid-specifications/raw/master/doc/img/liberapay-donate.png)](https://liberapay.com/RileyStarlight/donate)

Contributions are always welcome!

This is a **community-driven open source project**. Any contribution is highly appreciated, whether you are helping [fixing bugs or proposing new features](https://github.com/akornsys-rdi/pcbid-specifications/issues "Issues"), [pulling requests](https://github.com/akornsys-rdi/pcbid-specifications/pulls "Pull Requests"), improving the documentation or spreading the word.

Please :star: or watch this repository if this project helped you! You can also support this project on [Liberapay](https://liberapay.com/RileyStarlight/donate).

### ¿How to Pull Request to add new PCBIDs?

If you use PCBID for your own projects, please fork this repository, add your author and project data to the `projects.txt` file following the instructions present in these files and make a Pull Request with the changes.

## :scroll: Copyright & License

Copyright © 2020 RileyStarlight

### Documentation

[![CC BY SA 4.0](https://github.com/akornsys-rdi/pcbid-specifications/raw/master/doc/img/ccbysa-logo.png)](https://creativecommons.org/licenses/by-sa/4.0/ "Creative Commons BY-SA 4.0")

The documentation of this project is licensed under CC BY SA 4, see the [`LICENSE.BYSA4.txt`](LICENSE.BYSA4.txt) file for details.

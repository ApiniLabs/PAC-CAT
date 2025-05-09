# PAC-CAT

## `PAC-CAT` in a Nutshell

`PAC-CAT` augments the base [PAC-ID](https://github.com/ApiniLabs/pac-id) specification by defining
- how to structure the `identifier` for different entity categories. 
- shorthand rules for omitting `id segment key`s to reduce URL length
 
Example for a device:
```
HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/21:210263/8008:20230205/8009:ABC
```

## Introduction

`PAC-ID`s might be used to identify different categories of entities. Entities of different categories are treated differently (e.g. a substance can be aliquoted, while a device cannot; a method instructs a device what to do, while a run documents what was done).

While the basic specification for the `PAC-ID` has been intentionally kept minimal, `PAC-ID`s are much more powerful if they are issued both, systematically and with a some verbosity. This page contains recommendations how to structure the `identifier` of the `PAC-ID`. They are designed around these goals goals:

- Verbose enough so that it is always clear
  - to what entity the `PAC-ID` is pointing to
  - what the uniqueness scope is
- Reliable and easy for service discovery with [PAC-ID Resolver](https://github.com/ApiniLabs/pac-id-resolver)


## Specification

### Structure of the `identifier`
To add the required verbosity to `PAC-ID`s, the following structure for the `identifier` MUST be used, which is based on specific `id segments` that identify `categories`, that are followed by one or more known `id segment`s. The `id segment` identifying the start of such a `category`always starts with a `-`. 
The railroad diagram below illustrates the basic principle.

![Segment groups](images/pac-cat-identifier-structure-railroad.svg )


Example of a balance:
```
HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/21:210263/8008:20230205/8009:ABC
                             |category segments   |custom segments       |
                          ^ category key
```

The following chapters explain the composition in more detail.

### Identify the Category
In order to account for these differences the first `id segment` MUST indicate the category.

These categories MUST be used:
| `category key` | Main Category | Subcategory | Meaning |
| :---: | :---: | :---: | :--- |
| `-MD` | **M**(aterial) | **D**(evice) | **Device** (Or equipment, apparatus, appliance, instrument and the like) <br> _A **Device** is a uniquely identifiable item, non-aliquotable and not dividable._ |
| `-MS` | **M**(aterial) | **S**(ubstance) | **Substance** (Or source material, aliquot, sample, product and the like) <br> _A **Substance** is a uniquely identifiable item, aliquotable and/or dividable._ |
| `-MC` | **M**(aterial) | **C**(onsumable) | **Consumable** <br> _**Consumables** are typically bulk goods with limited lifespan. A **Consumable** is an item with a uniquely identifiable type and typically countable._ 
| `-MM` | **M**(aterial) | **M**(isc) | **Misc** <br> _Anything that doesn’t fit other material (`-M`) types - **ideally never used**._ |
| `-DR` | **D**(ata) | **R**(esult) | **Result** (Or completed run data, report, certificate of analysis (CoA) or the like) <br> _A **Result** is data that is a direct result of a completed run of a method (`-DM`)._ |
| `-DM` | **D**(ata) | **M**(ethod) | **Method** (Or run configuration, receipe, SOP and the like) <br> _A **Method** is a definition of a certain process or workflow._ |
| `-DC` | **D**(ata) | **C**(alibration) | **Calibration** (Or a basic configuration.) <br> _A **Calibration** is changeable data that is used as a basis for running a method (`-DM`) that creates progress data (`-DP`) and / or result data (`-DR`)._ |
| `-DP` | **D**(ata) | **P**(rogress) | **Progress** (Or status update, live data or the like) <br> _**Progress** data is of time-limited validity occurring while a method (`-DM`) is executed._ |
| `-DS` | **D**(ata) | **S**(tatic) | **Static** (Or metadata, datasheet, master data, physical properties or the like.) <br> _ **Static** data is unchangeable and universally true._ | 

### Segment Structure Within a Category 

#### `category segments`
Within a `category`, the `id segment key`s MUST follow this structure:
|[Category](#categories) | `id segment key`s |
|:---|:---|
**Materials**
Device | **`-MD` <br>`240` (Model number) <br> `21`  (Serial number)** <br>
Substance | **`-MS` <br> `240`  (Product number)** <br> `10`  (Batch number) <br> `20`  (Container size) <br> `21`  (Container number) <br> `250` (Aliquot)
Consumable |**`-MC` <br> `240`  (Product number)** <br> `10` (Batch number) <br> `20` (Packaging size) <br> `21` (Serial number) <br> `250`  (Aliquot)
Misc | **`-MM` <br> `240` (Product number)** <br> `10`  (Batch Number) <br> `20` (Packaging size) <br> `21` (Serial number) <br> `250` (Aliquot)
**Materials**
Result | **`-DR` <br> `21` (ID)** <br>
Method | **`-DM` <br> `21` (ID)** <br>
Calibration | **`-DC` <br> `21` (ID)** <br>
Progress | **`-DP` <br> `21` (ID)** <br>
Static | **`-DS` <br> `21` (ID)** <br>

`id segments key`s in **bold** MUST be used.
The other `id segment key`s SHOULD be added if they are available. 
The order SHOULD be preserved, even if optional `id segment`s are omitted.

#### `custom segments`
If needed, custom `id segement`s CAN be added. If so, they MUST be placed after the recommended `id segment`s.

### Short Notation
In oder to reduce the number of characters a short form MAY be used by omitting the `id segment key`s, like this:. 

```
HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/8008:20230205
```

The short notation omits the keys for segments of each category. Keys are implicitly assigned based on the [recommended segment order above](#recommened-segments-per-category) until an explicit key that differs is reached or an `id segment` starting with `-` is reached. Explicit keys can be used along implicit ones, as long as the order of segments is matched.

e.g. for ``HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/210263/8008:20230205``, `210263` is still regarded to have the implicit key `21`. For ``HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/8008:20230205/210263`` we can’t auto-assign a key for `210263` as it is preceded by a `id segment` with an explicit key. `210263` is therefore interpreted as a normal `id segment` without `id segment key`.

## Category Concatenation
Imagine a `PAC-ID` that points to a result set of a device. We’d usually want to know on which device that result was created. We CAN simply concatenate categories (in this case a material category to a data category):

Example:
```
HTTPS://PAC.METTORIUS.COM/-DR/240:123ABC/8008:20230205/-MD/240:BAL500/21:210263
                         | primary category           | additional category
```

The advantage of this is that it allows resolving device related attributes and services (e.g. device operation manual, …) via the same coupling table information entries also used for `PAC-ID`s relating to a device.

The category of the item the `PAC-ID` is referring to, SHALL be the first `category`.





## Terminology Used

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) "Key words for use in RFCs to Indicate Requirement Levels".

## FAQ

See [here](faq.md).

## License

Shield: [![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

This work is licensed under a
[Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg# 







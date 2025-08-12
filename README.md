# PAC-CAT

## `PAC-CAT` in a Nutshell

`PAC-CAT` augments the [PAC-ID](https://github.com/ApiniLabs/pac-id) specification by defining
- how to structure the `identifier` for different entity categories. 
- shorthand rules for omitting `id segment key`s to reduce URL length
- identify the issuing system 
 
Example for a device:
```
HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/21:210263/8008:20230205/8009:ABC
```

## Introduction
`PAC-ID`s might be used to identify different categories of entities. Entities of different categories are treated differently (e.g. a substance can be aliquoted, while a device cannot; a method instructs a device what to do, while a run documents what was done).

While the basic specification for the `PAC-ID` has been intentionally kept minimal, `PAC-ID`s are much more powerful if they are issued both, systematically and with a some verbosity. `PAC-CAT` specifies how to structure the `identifier` of the `PAC-ID`, to fulfil these goals:

- Verbose enough so that it is always clear
  - to what entity the `PAC-ID` is pointing to
  - what the uniqueness scope is
- Reliable and easy for service discovery with [PAC-ID Resolver](https://github.com/ApiniLabs/pac-id-resolver)

> [!IMPORTANT]
> PAC-CAT is not a data record - It is not the intention to fully describe an entity with these categorization. 


## Specification
> [!NOTE]
> While it is RECOMMENDED to use PAC-CAT to structure `PAC-ID`s, doing so is optional. A `PAC-ID` can be valid, without following this specification. 


### Structure of the `identifier`
The `PAC-ID`s `identifier` MUST be structured like this:
![Segment groups](images/pac-cat-identifier-structure-railroad.svg )

The first `id segment` MUST by a `category key`. 
The `category key` MUST start with a `-`, followed by the at least one letter.
`id segments` which are not `category key`s MUST NOT start with '-'.

Example of a balance:
```
HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/21:210263/8008:20230205/8009:ABC
                             |category segments   |custom segments       |
                          ^ category key
```

#### Concatenate a second category to identify the issuing system
Imagine a `PAC-ID` that points to a result set of a device. We’d usually want to know on which device that result was created. We CAN simply concatenate a second category (in this case a material category to a data category):

Example:
```
HTTPS://PAC.METTORIUS.COM/-DR/240:123ABC/8008:20230205/-MD/240:BAL500/21:210263
                         | primary category           | issuing system
```

The advantage of this is that it allows resolving device related attributes and services (e.g. device operation manual, …) via the same coupling table information entries also used for `PAC-ID`s relating to a device.

The category of the item the `PAC-ID` is referring to, SHALL be the first `category`.
The second category, if added, SHALL identify the issuing system.


### Predefined Categories
The following predefined categories MUST be used if applicable. 
Custom categories CAN be used if no suitable predefined category is available. _Use this as a last resort._

Mandatory `category segments`s are marked with * and in **boldface**. They MUST be used.
The other `category segments` SHOULD be added if they are available. 
The order SHOULD be preserved, even if optional `category segments` are omitted.

If needed, `custom segment`s CAN be added. If so, they MUST be placed after the recommended `category segments`.


#### Main Category *Materials*
Materials are physical entities, that can be uniquely identified.

|Subcategory | [`category key`](#category-key) | [`category segments`](#category-segments) |
|:--- | :------------: | :--- |
| **Device** (Or equipment, apparatus, appliance, instrument and the like)<br>*A Device is a uniquely identifiable item, non-aliquotable and not dividable.* | **`-MD`**| **`240` (Model&nbsp;code)** * <br> **`21` (Serial&nbsp;number)** *|
| **Substance** (Or source material, aliquot, sample, product and the like)<br>*A Substance is a uniquely identifiable item, aliquotable and/or dividable.*|**`-MS`**| **`240` (Product&nbsp;number)**&nbsp;* <br>`10` (Batch number)<br>`20` (Container size)<br>`21` (Container&nbsp;number)<br>`250` (Aliquot) |
| **Consumable**<br>*Consumables are typically bulk goods with limited lifespan. A Consumable is an item with a uniquely identifiable type and typically countable.*|**`-MC`** | **`240` (Product&nbsp;code)**&nbsp;*<br>`10` (Batch&nbsp;number)<br>`20` (Packaging size)<br>`21` (Serial&nbsp;number)<br>`250` (Aliquot)    |
| **Misc**<br>*Anything that doesn’t fit other material types – **ideally never used**.*|**`-MX`**| **`240` (Product&nbsp;code)**&nbsp;*<br>`10` (Batch&nbsp;number)<br>`20` (Packaging size)<br>`21` (Serial number)<br>`250` (Aliquot)    |

#### Main Category *Data*
Data refers to recorded information  — either digital or analog.

| | [`category key`](#category-key) | [`category segments`](#category-segments) |
|:--- | :------------: | :--- |
| **Result** (Or completed run data, report, certificate of analysis (CoA) or the like)<br>*A Result is data that is a direct result of a completed run of a method.*|**`-DR`**| **`21` (ID)**&nbsp;*|
| **Method** (Or run configuration, recipe, SOP and the like)<br>*A Method is a definition of a certain process or workflow.*|**`-DM`**| **`21` (ID)**&nbsp;*|
| **Calibration** (Or a basic configuration.)<br>*A Calibration is changeable data that is used as a basis for running a method that creates progress data and/or result data.* |**`-DC`**| **`21` (ID)**&nbsp;*|
| **Progress** (Or status update, live data or the like)<br>*Progress data is of time-limited validity occurring while a method is executed.*|**`-DP`**| **`21` (ID)**&nbsp;*|
| **Static** (Or metadata, datasheet, master data, physical properties or the like.)<br>*Static data is unchangeable and universally true.*|**`-DS`**| **`21` (ID)**&nbsp;*|
| **Misc** <br>*Anything that doesn’t fit other data types*– **ideally never used**|**`-DX`**| **`21` (ID)**&nbsp;*|

#### Main Category *Processors*
Processors are specific systems under the control of the issuer that assign and manage materials or data.


| | [`category key`](#category-key) | [`category segments`](#category-segments) |
|:--- | :------------: | :--- |
| **Software**<br>*Software are systems which generate, transform or store data.* <br><br>**Note**: Instruments often incorporate such functionality. In these cases it is RECOMMENDED to prioritize the material aspect of such instruments and use category `-MD`. | **`-PS`** | **`21`(Processor&nbsp;instance)**&nbsp;* <br> `240` (Processor&nbsp;code)|
| **Misc** <br>*Anything that doesn’t fit other processor types*– **ideally never used**| **`-PX`** | **`21`(Processor&nbsp;instance)**&nbsp;* <br> `240` (Processor&nbsp;code) |


### Short Notation
In oder to reduce the number of characters a short form MAY be used by omitting the `id segment key`s, like this:. 

```
HTTPS://PAC.METTORIUS.COM/-MD/BAL500/210263/8008:20230205
```

The short notation omits the keys for segments of each category. Keys are implicitly assigned based on the [recommended segment order above](#recommened-segments-per-category) until an explicit key that differs is reached or an `id segment` starting with `-` is reached. Explicit keys can be used along implicit ones, as long as the order of segments is matched.

e.g. for ``HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/210263/8008:20230205``, `210263` is still regarded to have the implicit key `21`. For ``HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/8008:20230205/210263`` we can’t auto-assign a key for `210263` as it is preceded by a `id segment` with an explicit key. `210263` is therefore interpreted as a normal `id segment` without `id segment key`.


### Examples: 
| **Entity Description** | **PAC-ID** | **Note** |
|------------------------|------------|----------|
| Production record managed in Fluidics360 ERP test instance at Mettorius | `HTTPS://PAC.METTORIUS.COM/-DR/21:12345/-PS/240:FLUIDICS360/21:TST` | `12345` is the ID assigned by the ERP; `TST` refers to the test instance. |
| Production record managed in the only Fluidics360 system at Mettorius | `HTTPS://PAC.METTORIUS.COM/-DR/21:12345/-PS/240:FLUIDICS360` | `12345` assigned by Fluidics360; single instance assumed, so `TST` omitted. |
| Instrument by Mettorius | `HTTPS://PAC.METTORIUS.COM/-MD/240:BAL500/21:12345/` | Mettorius produces instruments; adding an issuing system would not help with routing. |
| Calibration managed in the "ACME" tenant of "EosTec"`s SaaS system "Aurora" | `HTTPS://PAC.EOSTEC.COM/-DC/21:12345/-PS/240:AURORA/21:ACME` | `12345` assigned by Aurora; `ACME` is the tenant name. |
| Pencil used at "ACME". For stationery, they use an Excel-based asset list on ShareDot | `HTTPS://PAC.ACME.COM/-MC/240:EDELWEISS-3B/21:1234/-P/240:SHAREDOT/21:ASSETS.XLS` | `1234` is the ID given in `ASSETS.XLS` |
| Beehive of the ACME company. Tracked in Fluidics360 Asset Management | `HTTPS://PAC.ACME.COM/-MD/240:BEEHIVE/21:1234/-P/240:FLUIDICS360` | `1234` is the asset number assigned by SAP. |
| Result generated by BAL-500 balance with serial X78767 | `HTTPS://PAC.METTORIUS.COM/-DR/21:1234/-MD/240:BAL-500/21:X78767` | `1234` is the result ID; the issuing system is the balance itself. |
| Result managed by Mettorius LabCross software | `HTTPS://PAC.METTORIUS.COM/-DR/21:1234/-PS/240:LABCROSS` | `1234` is the result ID assigned by LabCross. |




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







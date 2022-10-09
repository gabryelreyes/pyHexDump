# pyHexDump <!-- omit in toc -->
A CLI tool written in Python to dump binary files and files in intel hex format. It can generate a report for any file based on report template. This is useful for images which contain specific data always on the same address, e.g. a CRC, signature, etc.

There are a lot of hex viewers already, but I was not able to find one which I could configure in a way to generate something like a report.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](http://choosealicense.com/licenses/mit/)
[![Repo Status](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)
![CI Status](https://github.com/BlueAndi/pyHexDump/actions/workflows/pylint.yml/badge.svg)

- [Installation](#installation)
- [Usage](#usage)
- [Examples](#examples)
  - [Dump data as 8-bit](#dump-data-as-8-bit)
  - [Dump data as 32-bit little endian](#dump-data-as-32-bit-little-endian)
  - [Print configuration](#print-configuration)
  - [Print report with template](#print-report-with-template)
  - [Configuration using structures](#configuration-using-structures)
  - [Define structure as datatype](#define-structure-as-datatype)
- [Macros](#macros)
  - [macros_compare_values()](#macros_compare_values)
  - [m_read_u8()](#m_read_u8)
  - [m_read_u16le()](#m_read_u16le)
  - [m_read_u16be()](#m_read_u16be)
  - [m_read_u32le()](#m_read_u32le)
  - [m_read_u32be()](#m_read_u32be)
  - [m_calc_checksum()](#m_calc_checksum)
  - [m_swap_bytes_u16()](#m_swap_bytes_u16)
  - [m_swap_bytes_u32()](#m_swap_bytes_u32)
  - [m_swap_words_u32()](#m_swap_words_u32)
- [FAQ](#faq)
  - [How to get a element in decimal in the template?](#how-to-get-a-element-in-decimal-in-the-template)
- [Issues, Ideas And Bugs](#issues-ideas-and-bugs)
- [License](#license)
- [Contribution](#contribution)

# Installation
```cmd
$ git clone https://github.com/BlueAndi/pyHexDump.git
$ cd pyHexDump
$ python setup.py install
```

# Usage

Show help information:
```cmd
$ pyHexDump --help
```

# Examples

## Dump data as 8-bit

```$ pyHexDump dump ./test/aurix_tc397.hex -a 0x80000020```

Result:
```
80000020: 02 58 DA 01 9B 1F 00 F0 0F 4F 10 F0 6C 41 C5 FF
80000030: 00 01 BC F1 82 04 6D 00 2F 04 91 30 00 FF 39 FF
80000040: 30 06 16 0F 4B 0F 41 F1 4B F2 51 F0 3C 01 3C 01
80000050: 4B 0F 31 F1 3B 80 3E 00 4B 0F 01 02 E2 08 3C 01
```

## Dump data as 32-bit little endian

```$ pyHexDump dump ./test/aurix_tc397.hex -a 0x80000020 -dt u32le```

Result:
```
80000020: 01DA5802 F0001F9B F0104F0F FFC5416C
80000030: F1BC0100 006D0482 3091042F FF39FF00
80000040: 0F160630 F1410F4B F051F24B 013C013C
80000050: F1310F4B 003E803B 02010F4B 013C08E2
80000060: 4800800B 00CE006D 4F409000 80DA4802
80000070: 5008003B F440F5A6 006D8402 01DA028B
80000080: F0108F0F 9000F16C FFFFFFFF 7FFFFFFF
80000090: F0248160 00873802 FFFFFFFF F0248164
800000A0: 0000D066 FFFFFFFF F0248168 00073802
800000B0: FFFFFFFF F024816C 00009826 FFFFFFFF
800000C0: F0248124 000000C9 FFFFFFFF F0248108
800000D0: 30360001 FFFFFFFF F024810C 0B690708
800000E0: FFFFFFFF F0248128 0121048E FFFFFFFF
800000F0: 00000000 FFFFFFFF FFFFFFFF FFFFFFFF
80000100: F8000091 3048FFD9 0200000D 0FDCF402
80000110: 00000000 00000000 00000000 00000000
```

## Print configuration

```$ pyHexDump print ./test/aurix_tc397.hex ./test/config.json```

with ```config.json``` like
```json
{
    "elements": [{
        "name": "UCB00_BMI_BMHDID",
        "addr": "0xAF400000",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_STAD",
        "addr": "0xAF400004",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_CRCBMHD",
        "addr": "0xAF400008",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_CRCBMHD_N",
        "addr": "0xAF40000C",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_PWx",
        "addr": "0xAF400104",
        "dataType": "u32le",
        "count": 8
    }, {
        "name": "UCB00_CONFIRMATION",
        "addr": "0xAF4001F0",
        "dataType": "u32le",
        "count": 1
    }]
}
```

Result:
```
UCB00_BMI_BMHDID @ AF400000: B35900FE
UCB00_STAD @ AF400004: A0000000
UCB00_CRCBMHD @ AF400008: 31795570
UCB00_CRCBMHD_N @ AF40000C: CE86AA8F
UCB00_PWx @ AF400104: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
UCB00_CONFIRMATION @ AF4001F0: 43211234
```

## Print report with template
The [Mako template library](https://www.makotemplates.org/) is used, to provide a lot of functionality. Please have a look to the [Mako documentation](https://docs.makotemplates.org/en/latest/) for details.

```$ pyHexDump print ./test/aurix_tc397.hex ./test/config.json --template ./test/markdown.mao```

with ```config.json``` like
```json
{
    "elements": [{
        "name": "UCB00_BMI_BMHDID",
        "addr": "0xAF400000",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_STAD",
        "addr": "0xAF400004",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_CRCBMHD",
        "addr": "0xAF400008",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_CRCBMHD_N",
        "addr": "0xAF40000C",
        "dataType": "u32le",
        "count": 1
    }, {
        "name": "UCB00_PWx",
        "addr": "0xAF400104",
        "dataType": "u32le",
        "count": 8
    }, {
        "name": "UCB00_CONFIRMATION",
        "addr": "0xAF4001F0",
        "dataType": "u32le",
        "count": 1
    }]
}
```

with ```markdown.mao``` like
```mako
<%text># Aurix TC397 - Blinky Example</%text>

<%text>## User Control Block 00</%text>

|Short Name|Value|
|----------|-----|
| BMI_BMHDID | ${UCB00_BMI_BMHDID} |
| STAD | ${UCB00_STAD} |
| CRCBMHD | ${UCB00_CRCBMHD} |
| CRCBMHD_N | ${UCB00_CRCBMHD_N} |
| PWx | ${UCB00_PWx} |
| CONFIRMATION | ${UCB00_CONFIRMATION} |
<%
    bmi_bmhdid = int(UCB00_BMI_BMHDID, 16)
    bmi    = (bmi_bmhdid >>  0) & 0xFFFF
    bmhdid = (bmi_bmhdid >>  16) & 0xFFFF
    pindis = (bmi >> 0) & 0x01
    hwcfg  = (bmi >> 1) & 0x07

    mode_by_hwcfg = "disabled"
    if pindis == 0:
        mode_by_hwcfg = "enabled"

    start_up_mode = "invalid"
    if hwcfg == 0x07:
        start_up_mode = "internal start from flash"
    elif hwcfg == 0x06:
        start_up_mode = "alternate boot mode"
    elif hwcfg == 0x04:
        start_up_mode = "generic bootstrap loader mode"
    elif hwcfg == 0x03:
        start_up_mode = "asc bootstrap loader mode"
    
    is_bmh_valid = "invalid"
    if bmhdid == 0xB359:
        is_bmh_valid = "OK"
    
    calculated_crc_bmhd = m_calc_checksum("u32le", 0xAF400000, 0xAF400008, 0x04c11db7, 32, 0xffffffff, True, True, True)
    calculated_crc_bmhd_n = m_calc_checksum("u32le", 0xAF400000, 0xAF400008, 0x04c11db7, 32, 0xffffffff, True, True, False)

    is_bmh_integrity_given = "Not OK"
    if calculated_crc_bmhd == int(UCB00_CRCBMHD, 16):
        if calculated_crc_bmhd_n == int(UCB00_CRCBMHD_N, 16):
            is_bmh_integrity_given = "OK"
%>
<%text>### Boot Mode Index (BMI)</%text>
* Mode selection by configuration pins: ${mode_by_hwcfg}
* Start-up mode: ${start_up_mode}

<%text>### Boot Mode Header Identifier (BMHDID)</%text>
Is boot mode header valid: ${is_bmh_valid}

<%text>### Boot Mode Header CRC (CRCBMHD/CRCBMHD_N)</%text>
Is boot mode header integrity given: ${is_bmh_integrity_given}
```

Result:
```markdown
# Aurix TC397 - Blinky Example

## User Control Block 00

|Short Name|Value|
|----------|-----|
| BMI_BMHDID | B35900FE |
| STAD | A0000000 |
| CRCBMHD | 31795570 |
| CRCBMHD_N | CE86AA8F |
| PWx | 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000 |
| CONFIRMATION | 43211234 |

### Boot Mode Index (BMI)
* Mode selection by configuration pins: enabled
* Start-up mode: internal start from flash

### Boot Mode Header Identifier (BMHDID)
Is boot mode header valid: OK

### Boot Mode Header CRC (CRCBMHD/CRCBMHD_N)
Is boot mode header integrity given: OK
```

## Configuration using structures
If several elements are right behind each other like in a structure, it can be configured in a similar way by using a list of elements for a datatype. The address of each element in the structure is calculated by the given base address in the datatype of each element.

Note that nested structures are not supported yet!

```json
{
    "elements": [{
        "name": "UCB00",
        "addr": "0xAF400000",
        "dataType": [{
            "name": "BMI_BMHDID",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "STAD",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "CRCBMHD",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "CRCBMHD_N",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "PWx",
            "dataType": "u32le",
            "offset": "0x0104",
            "count": 8
        }, {
            "name": "CONFIRMATION",
            "dataType": "u32le",
            "offset": "0x01F0",
            "count": 1
        }],
        "count": 1
    }]
}
```

To access it in the template, you can use the "." notation or Python dictionary syntax.

with ```markdown.mao``` like
```mako
<%text># Aurix TC397 - Blinky Example</%text>

<%text>## User Control Block 00</%text>

|Short Name|Value|
|----------|-----|
| BMI_BMHDID | ${UCB00.BMI_BMHDID} |
| STAD | ${UCB00.STAD} |
| CRCBMHD | ${UCB00.CRCBMHD} |
| CRCBMHD_N | ${UCB00.CRCBMHD_N} |
| PWx | ${UCB00.PWx} |
| CONFIRMATION | ${UCB00.CONFIRMATION} |
<%
    bmi_bmhdid = int(UCB00.BMI_BMHDID, 16)
    bmi    = (bmi_bmhdid >>  0) & 0xFFFF
    bmhdid = (bmi_bmhdid >>  16) & 0xFFFF
    pindis = (bmi >> 0) & 0x01
    hwcfg  = (bmi >> 1) & 0x07

    mode_by_hwcfg = "disabled"
    if pindis == 0:
        mode_by_hwcfg = "enabled"

    start_up_mode = "invalid"
    if hwcfg == 0x07:
        start_up_mode = "internal start from flash"
    elif hwcfg == 0x06:
        start_up_mode = "alternate boot mode"
    elif hwcfg == 0x04:
        start_up_mode = "generic bootstrap loader mode"
    elif hwcfg == 0x03:
        start_up_mode = "asc bootstrap loader mode"
    
    is_bmh_valid = "invalid"
    if bmhdid == 0xB359:
        is_bmh_valid = "OK"

    calculated_crc_bmhd = m_calc_checksum("u32le", 0xAF400000, 0xAF400008, 0x04c11db7, 32, 0xffffffff, True, True, True)
    calculated_crc_bmhd_n = m_calc_checksum("u32le", 0xAF400000, 0xAF400008, 0x04c11db7, 32, 0xffffffff, True, True, False)

    is_bmh_integrity_given = "Not OK"
    if calculated_crc_bmhd == int(UCB00.CRCBMHD, 16):
        if calculated_crc_bmhd_n == int(UCB00.CRCBMHD_N, 16):
            is_bmh_integrity_given = "OK"
%>
<%text>### Boot Mode Index (BMI)</%text>
* Mode selection by configuration pins: ${mode_by_hwcfg}
* Start-up mode: ${start_up_mode}

<%text>### Boot Mode Header Identifier (BMHDID)</%text>
Is boot mode header valid: ${is_bmh_valid}

<%text>### Boot Mode Header CRC (CRCBMHD/CRCBMHD_N)</%text>
Is boot mode header integrity given: ${is_bmh_integrity_given}

```

## Define structure as datatype
If a structure shall be used several times, define it as a datatype and use its name.

```json
{
    "elements": [{
        "name": "UCB00",
        "addr": "0xAF400000",
        "dataType": "UCB_t",
        "count": 1
    }],
    "structures": [{
        "name": "UCB_t",
        "elements": [{
            "name": "BMI_BMHDID",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "STAD",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "CRCBMHD",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "CRCBMHD_N",
            "dataType": "u32le",
            "count": 1
        }, {
            "name": "PWx",
            "dataType": "u32le",
            "offset": "0x0104",
            "count": 8
        }, {
            "name": "CONFIRMATION",
            "dataType": "u32le",
            "offset": "0x01F0",
            "count": 1
        }]
    }]
}
```

# Macros

The following macros are available in the templates.

## macros_compare_values()
Compares the set value with the actual value.

Parameters:
* set_value: Set value
* actual_value: Actual value
* value_format="{:02X}": Value format used to print them in case they are different.

Returns:
* "Ok": If both values are equal.
* "Not Ok (Set: &lt;set_value&gt;, Actual: &lt;actual_value&gt;)": If the values are different.

## m_read_u8()
Read unsigned 8-bit value from binary data at given address and returns it.

Parameters:
* addr: Address

## m_read_u16le()
Read unsigned 16-bit little endian value from binary data at given address and returns it.

Parameters:
* addr: Address

## m_read_u16be()
Read unsigned 16-bit big endian value from binary data at given address and returns it.

Parameters:
* addr: Address

## m_read_u32le()
Read unsigned 32-bit little endian value from binary data at given address and returns it.

Parameters:
* addr: Address

## m_read_u32be()
Read unsigned 32-bit big endian value from binary data at given address and returns it.

Parameters:
* addr: Address

## m_calc_checksum()

Parameters:
* binary_data_endianess: The binary data endianess and bit width:
    * "u8": unsigned 8-bit
    * "u16le": unsigned 16-bit little endian
    * "u16be": unsigned 16-bit big endian
    * "u32le": unsigned 32-bit little endian
    * "u32be": unsigned 32-bit big endian
* start_address: Start address of the CRC calculation.
* end_address: End address of the CRC calculation (not included).
* polynomial: The polynomial for the CRC calculation.
* bit_width: The bit width, e.g. 8 in case of a CRC-8.
* seed: The seed value which to use.
* reverse_input: If the input data shall be reflected, set to True otherwise to False.
* reverse_output: If the output data shall be reflected, set to True otherwise to False.
* final_xor: If the output shall be have a final XOR with all bits set, set to True otherwise to False.

## m_swap_bytes_u16()
Swaps the bytes of a unsigned 16-bit value.

Parameters:
* value: Source value

Returns:
* Swapped value

## m_swap_bytes_u32()
Swaps the bytes of a unsigned 32-bit value.

Parameters:
* value: Source value

Returns:
* Swapped value

## m_swap_words_u32()
Swaps the 16-bit words of a unsigned 32-bit value.

Parameters:
* value: Source value

Returns:
* Swapped value

# FAQ

## How to get a element in decimal in the template?
Define the following expression filter in your template:
```mako
<%!
    def toDec(hex_value_str):
        return str(int(hex_value_str, 16))
%>
```

By using this filter, the value is shown in decimal.
```mako
${hex_value | toDec}
```

Note, a expression filter gets a string as parameter and expects to get a string returned!

# Issues, Ideas And Bugs
If you have further ideas or you found some bugs, great! Create a [issue](https://github.com/BlueAndi/pyHexDump/issues) or if you are able and willing to fix it by yourself, clone the repository and create a pull request.

# License
The whole source code is published under the [MIT license](http://choosealicense.com/licenses/mit/).
Consider the different licenses of the used third party libraries too!

# Contribution
Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, shall be licensed as above, without any additional terms or conditions.

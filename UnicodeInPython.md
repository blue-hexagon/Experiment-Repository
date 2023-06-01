# Unicode In Python
```python
import re
import sys
from pprint import pprint

import colorama
from colorama import Fore, Back

colorama.init(autoreset=True)

if __name__ == '__main__':
    with open("./_test_unicode.py", 'r', encoding='utf-8') as f:
        file_content = f.readlines()
        file_content = ''.join(str(s) for s in file_content)
        matches = re.findall(pattern=r'\s+((integer|bin|bytes|string|hex|oct)_[a-z_]+)(?:\s+=\s+)([a-zA-Z0-9[\]*()\'\\.,= ]+)', string=file_content)
    var_expression_map = dict()
    for match in matches:
        var_expression_map[match[0].strip(' ')] = match[2]
    r"""
    There are up to six ways that Python will allow you to type the same Unicode character.
    ---------------------------------------------------------------------------------------
    Escape Sequence     Meaning                         How To Express "a"
    "\ooo"              Character with octal value ooo                      "\141"
    "\xhh"              Character with hex value hh                         "\x61"
    "\N{name}"          Character named name in the Unicode database        "\N{LATIN SMALL LETTER A}"
    "\uxxxx"            Character with 16-bit (2-byte) hex value xxxx       "\u0061"
    "\Uxxxxxxxx"        Character with 32-bit (4-byte) hex value xxxxxxxx   "\U00000061"
    """
    unicode_notation = (
            "a" ==
            "\x61" ==
            "\N{LATIN SMALL LETTER A}" ==
            "\u0061" ==
            "\U00000061"
    )  # This expression is true as these  are all equivalent.

    # string = "Hello ρythθn! 🐍"

    string = "Hello YouTube"
    string_to_ascii = [ascii(s) for s in string]
    string_to_bits = [bin(ord(s)) for s in string]
    string_to_int = [(ord(s)) for s in string]
    string_to_bytes = bytes(string.encode('utf-8'))
    string_to_hex = [hex(ord(s)) for s in string]
    string_to_oct = [oct(ord(s)) for s in string]

    integer = 65534
    integer_to_bits = bin(integer)
    integer_to_string = str(integer)
    integer_to_ascii = ascii(integer)
    integer_to_bytes = integer.to_bytes(length=4, byteorder='big')  # byteorder => 'little' | 'big'
    integer_to_hex = hex(integer)
    integer_to_oct = oct(integer)

    bytess = b'Hello \xcf\x81yth\xce\xb8n! \xf0\x9f\x90\x8d'  # Equivalent to: bytes("Hello ρythθn! 🐍".encode('utf-8))
    bytess_to_bits = [bin(byte) for byte in bytess]
    # bytes_to_string = str(bytes_base.decode('utf-8'))
    # bytes_to_ascii = ascii(bytes)
    # bytes_to_integer = bytes.to_bytes(length=4, byteorder='big')  # byteorder => 'little' | 'big'
    # bytes_to_hex = hex(bytes)
    # bytes_to_oct = oct(bytes)

    previous_group = ""
    current_group = ""
    groups = ['integer', 'bytes', 'string', 'bin', 'hex', 'oct']
    disallowed_vars = ['previous_group', 'current_group', 'groups', 'disallowed_vars', 'Back', 'Fore', 'colorama', 'unicode_notation', 'var_expression_map', 'sys', 're', 'pprint', 'match',
                       'matches', 'file_content', 'f']
    for var in dir():
        if var[0:2] == '__' or var in disallowed_vars:
            continue
        else:
            if var in groups:
                previous_group = current_group
                current_group = var
                if previous_group != current_group:
                    print(Fore.YELLOW + "-" * 16 + Fore.RED + current_group.title() + Fore.YELLOW + "-" * 16 + Fore.WHITE)

            previous_group = var
            if "_" not in var:
                print(Fore.MAGENTA + var + " =  " + str(eval(var)) + Fore.WHITE)

            else:
                # if "bytess" in var:
                #     var = var.replace("bytess", "bytes")
                print(Fore.GREEN + var + ":" + Fore.WHITE, end='')
                try:
                    mapped_var = var_expression_map.get(var)
                    print(Fore.YELLOW + mapped_var + Fore.WHITE)
                except Exception:
                    print(Fore.YELLOW + "..." + Fore.WHITE)
                    # pprint(var_expression_map)
                    # print(var_expression_map.get(var))
                    # print(var)
                    # sys.exit(0)
                print(Fore.BLUE + str(eval(var)) + Fore.WHITE)
```

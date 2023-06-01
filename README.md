# Experiments
Various small programs or research on various matters

## FileStego
Simple program that encodes and decodes a Python payload as harddrive files.
```python
import glob
import os
import platform
import random
import re
import sys
from os import walk
from typing import List, Tuple


class FileMetadata:
    def __init__(self):
        ...


class File:
    def __init__(self, ascii_char: str, rotation: str, string_index: str):
        self.ascii_char = ascii_char
        self.string_index = string_index
        self.rotation = rotation


class Payload:
    def __init__(self, raw_content: str) -> None:
        self.raw_content: str = raw_content
        random.seed("1")
        self.rotation_sequence = [random.randint(0, len(raw_content)) for _ in range(len(raw_content))]
        # print(self.raw_content)
        # print(self.rotation_sequence)

        coded_content = ""
        for idx in range(len(self.rotation_sequence)):
            raw_content_char = bytes(self.raw_content[idx].encode('utf-16'))
            print(raw_content_char)
            rotation_sequence_int = self.rotation_sequence[idx]
            if raw_content_char not in ['\r', '\n']:
                coded_content += chr(ord(raw_content_char) + rotation_sequence_int)
        print(coded_content)
        decoded_conent = ""
        for idx in range(len(self.rotation_sequence)):
            enc_content_char = coded_content[idx]
            print(enc_content_char)
            rotation_sequence_int = self.rotation_sequence[idx]
            if coded_content[idx] not in ['\r', '\n']:
                decoded_conent += chr(ord(enc_content_char) - rotation_sequence_int)
        print(decoded_conent)
        sys.exit(0)


class FileEncoder:
    write_files: List[File] = []

    def write(self, python_script: str) -> None:
        for idx, char in enumerate(python_script):
            idx = str(idx).rjust(4, "0")
            rotation = random.randint(0, 25)
            self.write_files.append(File(ascii_char=str(ord(char)), rotation=str(rotation), string_index=f"{idx}"))
        for file in self.write_files:
            with open(f"dist/{int(file.ascii_char) + int(file.rotation)}.{file.string_index}", mode="wb+") as f:
                rot = str("SHIFT" + str(file.rotation))
                f.write(bytes(str(rot).encode("utf-8")))


class FileDecoder:
    read_files: List[Tuple[int, int]] = []  # List[Tuple[ascii_character, str_index]]

    def decode(self) -> str:
        files = sorted(self.read_files, key=lambda file: file[1][1:])
        rendered_string = ""
        for file in files:
            with open(f"./dist/{file[0]}.{file[1]}", mode="r") as f:
                rotation = f.read()
            rendered_string += chr(int(file[0]) - int(rotation[5:]))
        return rendered_string

    def read(self) -> None:
        for (dirpath, dirnames, filenames) in walk("./dist"):
            for filename in filenames:
                self.read_files.append(tuple(filename.split(".")))


class Executor:
    def __init__(self):
        self.check_os_is_supported()
        self.payload = """import subprocess\r\nprocess = subprocess.Popen(('python -c "import this"'), shell=True, stdout=subprocess.PIPE)\r\nprint('The flag is: '+ process.stdout.read()[4:7].decode('utf-8'))"""
        self.encoder = FileEncoder()
        self.decoder = FileDecoder()

    @staticmethod
    def check_os_is_supported() -> None:
        if not bool(re.match("windows", str(platform.platform()).lower())):
            sys.exit("This application only supports Windows OS")

    def encode(self):
        self.cleanup()
        payload = Payload(self.payload)
        self.encoder.write(payload.raw_content)

    def decode(self):
        self.decoder.read()
        deciperhed_string = self.decoder.decode()
        exec(deciperhed_string)

    @staticmethod
    def cleanup() -> None:
        files = glob.glob("./dist/*")
        for f in files:
            os.remove(f)


if __name__ == "__main__":
    executor = Executor()
    executor.encode()
    executor.decode()
```

## Test Unicode In Python
Testing unicode and the conversion from/to various primitive (+ string) datatypes.

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

### Program Output
![image](https://github.com/blue-hexagon/Experiments/assets/26361520/388abcfb-caf0-4848-afdf-aa5b73dda6df)

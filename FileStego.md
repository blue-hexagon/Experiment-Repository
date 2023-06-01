# FileStego

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

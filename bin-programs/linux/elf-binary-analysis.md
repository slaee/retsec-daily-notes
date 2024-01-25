Tools:
`gcc` to make ELF.
`readelf` to parse the ELF header.
`objdump` to parse the ELF header and disassemble the source code.
`nm` to view ELF's symbols.
`patchelf` to change some ELF properties.
`objcopy` to swap out ELF sections.
`strip` to remove otherwise-helpful informations (such as symbols). Makes binary `stripped` and shows `stripped`  when checking a binary info with `file` command.
`kaitai struct` [kaitai io](https://ide.kaitai.io/) to look through ELF interactively.


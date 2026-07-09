# altium-bms

Altium Designer source files for the MAX17320-based 2S4P battery
management board this repo's firmware (`../max17320.c` etc.) targets.

## Files

- `0001.SchDoc` -- schematic.
- `max17320g_2.PcbDoc` -- PCB layout.
- `Batterie_Überwachung.PrjPcb` / `.PrjPcbStructure` -- project files.
- `Batterie_Überwachung.SCHLIB` -- project schematic library.
- `Batterie_Überwachung.BomDoc` -- bill of materials.
- `Batterie_Überwachung.OutJob` -- output job configuration.
- `Batterie_Überwachung.pdf` -- rendered schematic (view without Altium).
- `Components/` -- custom component library (schematic symbols, PCB
  footprints, compiled IntLib) authored for this project.

Not included: build history/backups, generated "Project Outputs", PCB
fab order confirmation, or third-party vendor component libraries
(datasheets/3D models pulled from component library services) -- those
are either not reproducible source, not this project's own content, or
not meant for a public repo.

Current sense resistor on this board: **MFC0603-R005FT5, 5.0mOhm**
(referenced by `MAX17320_RSENSE_MOHM` in `../max17320_config.h`).
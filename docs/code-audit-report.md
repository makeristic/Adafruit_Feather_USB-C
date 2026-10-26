# USB-C Feather template correctness audit

Date: 2026-10-26. Scope: legacy schematic, cached symbols, PCB, project settings, template metadata, README installation, and existing licence. KiCad CLI nightly 10.99.0 checks ran on temporary copies; original design data was retained.

## Corrected documentation and metadata

The README previously described USB-C VBUS as unconnected. Netlist export proves that VBUS pins A4/A9/B4/B9 connect to the VBUS header net, with ground and shield on GND. The README now distinguishes those assigned power nets from the unfinished CC/data wiring. [meta/info.html](../meta/info.html) now describes an unrouted USB-C template, fixes the original description typo, includes image alternative text, and retains the original copyright and upstream licence. Installation guidance explicitly preserves the cached USB-C symbol.

## Findings requiring a completed design

- **P1 — This is an incomplete USB-C/electrical design.** [Adafruit_Feather_USB-C.kicad_pcb](../Adafruit_Feather_USB-C.kicad_pcb#L6) has zero tracks. The [cached connector definition](../Adafruit_Feather_USB-C-cache.lib#L149) and schematic leave CC1, CC2, four USB data pins, and both SBU pins unconnected; there are no CC termination resistors, controller, regulator, or charging circuit. ERC reports **37 findings: 12 errors** (four undriven power inputs and eight unconnected connector pins) **and 25 warnings** (24 isolated labels and one unavailable legacy library-symbol name). Required connections versus deliberate no-connects depend on the completed circuit and must be resolved there.
- **P2 — Modern manufacturing checks are not clean.** DRC using default nightly constraints and current standard footprint libraries reports **58 violations** (30 copper-edge clearance errors, 11 solder-mask bridge errors, seven silk-edge warnings, eight library-footprint mismatch warnings, and two silk-over-copper warnings), plus **10 unrouted connections**. Header copper has 0.42 mm edge clearance against the tested 0.5 mm constraint; the USB connector also needs mask/silkscreen review. Actual fabrication constraints and connector drawings must guide completion; rules were not weakened to clear findings.
- **P2 — Nominal dimensions do not guarantee mechanical compatibility.** [Header placement](../Adafruit_Feather_USB-C.kicad_pcb#L251) has 2.54 mm pitch and 20.32 mm row separation. The main rounded body is 50.8 × 22.86 mm, while the [USB-C extension](../Adafruit_Feather_USB-C.kicad_pcb#L519) reaches 1.651 mm beyond the nominal USB-side edge. The [mounting footprints](../Adafruit_Feather_USB-C.kicad_pcb#L427) use two 2.5 mm USB-side holes and two 2.2 mm far-side holes, with far-side centres shifted outward by 0.635 mm. The [Feather specification](https://learn.adafruit.com/adafruit-feather/feather-specification) describes 2.54 mm corner holes and discourages extending the USB side. Review these details for the intended Wing/enclosure; no mechanical geometry was silently redesigned.
- **P2 — Legacy assets need deliberate migration.** The symbol name `Connector:USB_C_Receptacle_USB2.0` is absent in the tested current library, but its original definition is supplied in the cache and successfully exports. Current footprints differ from the embedded 2020 copies. Four model paths use `${KISYS3DMOD}` and historical `.wrl` names absent from the tested nightly installation. A modern library refresh/conversion and 3D-alignment review remain separate engineering work.

## Checks and evidence

The schematic exported a one-page SVG and an XML netlist with four connectors. A separate comparison grouped every connector pin by net in the schematic and PCB: **all 36 pin/net groups matched**, including all four VBUS pins, ground/shield, battery polarity, and header labels. Every assigned footprint name exists in the installed standard footprint libraries. Mechanical dimensions were read from source coordinates; connector dimensions were not recertified against manufacturer drawings.

Commands ran against a temporary copy under `/tmp/makeristic-library-template-audit/`, with `XDG_CONFIG_HOME` pointing to a temporary `config` directory. Standard symbol/footprint tables were configured only in that temporary environment:

```sh
kicad-cli-nightly sch export netlist --format kicadxml --output netlist.xml Adafruit_Feather_USB-C.sch
kicad-cli-nightly sch export svg --output schematic-svg Adafruit_Feather_USB-C.sch
kicad-cli-nightly sch erc --format json --output erc.json Adafruit_Feather_USB-C.sch
kicad-cli-nightly pcb drc --format json --output drc.json Adafruit_Feather_USB-C.kicad_pcb
```

The output directory was created before SVG export. Direct CLI schematic upgrade and built-in schematic-parity DRC rejected the legacy `.sch` format (`Expecting '('`); they are not claimed as passing. The netlist comparison above used the successful legacy export instead. No converted schematic or generated audit artifacts were committed.

Relative documentation links, template image paths, and `git diff --check` passed. ERC/DRC reports contain the findings above; command completion is not a clean design sign-off. Physical assembly, USB behaviour, and manufacturing were not tested.

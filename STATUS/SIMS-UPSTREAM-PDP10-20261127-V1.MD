# SIMS upstream PDP-10 integration (20261127-v1)

Original upstream: rcornwell/sims, master at 18b6343.
Local baseline: abb7ee6. Integrated master: 348106f.

Eight non-merge upstream patches were cherry-picked with original authorship
and upstream references, in dependency order, without conflicts:

| Upstream | Local | Scope |
| --- | --- | --- |
| 2650e42 | 380ab54 | KL10 NIA DECnet receive fixes |
| a83fd0f | 71cf2d7 | TYM zero-size division guard |
| 813f838 | f328e50 | TYM whitespace |
| da1efc5 | f93a7d1 | DC10 disconnected-line handling |
| 8b7d4a9 | 113d3e9 | DDC unattached-device handling |
| 35bec23 | a8db42f | DDC function codes |
| 101af59 | e18e813 | PiDP-10 MI update |
| be8cd4e | 348106f | KI memory-access correction |

Validation on Termux: optimized C99 build with -Werror, SDL2 and networking
support for the applicable targets. All five targets (pdp6, pdp10-ka,
pdp10-ki, pdp10-kl, pdp10-ks) built and passed RegisterSanityCheck.
Existing daimos-testkit validation scripts check-simh-pdp6-go-pc-v155.sh
and daimos-simh-console-pc-v115.sh passed against the new PDP-6 binary.
These checks cover compilation, register sanity and PDP-6 console behavior;
they do not establish device-specific DECnet, TYM, DC10, DDC, KI memory,
or physical PiDP-10 behavioral acceptance. No DAIMOS kernel code changed.

The shared SCP/display update 18b6343 overlaps local changes and remains
unintegrated. The remaining non-equivalent upstream changes concern that
update, I7000 and SIM_CARD comments; these were not part of this PDP-10 pass.

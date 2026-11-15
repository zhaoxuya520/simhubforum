# DITO home identity recovery

Reference: https://github.com/maow318/simhubforum/releases/tag/v29.3

The reference repository publishes iOS binaries, not the implementation source. Its release notes explicitly require reading the card once while it has its real home 515-66 identity. The standard IPA's SHA-256 is `1d0b3c0bea93695bf467efda8c3b07793b03532fd55c096207177e8f2edf2b5d`.

Inspection of that unencrypted arm64 executable confirms:

- `DitoHomeIdentityStore` persists the full, actually read home IMSI by ICCID, under `vowifi.dito.home_imsi.`. The save is read back and checked.
- The strict DITO issuer prefix is `8963662600`; accepted ICCIDs have 19 or 20 ASCII digits. A home IMSI has 14 or 15 ASCII digits, begins `51566`, and has a two-digit MNC. The validator starts at `0x100206364`.
- On a `20404` roaming identity, the resolver restores only the same ICCID's saved home IMSI. It creates a temporary identity with MNC length 2, preserving the remaining card metadata. Without a valid home record, Wi-Fi Calling is rejected with an explicit home-identity-required message. Restore entry points start at `0x1001800fc` and `0x100206488`; the save routine starts at `0x10017fe90`.
- It does not derive an original home subscriber number by replacing an IMSI prefix. Its strings explicitly say that a roaming IMSI cannot supply the missing home identity. Airplane mode before removal is advice for deregistration, not a way to recover a missing identity.

The reference's DITO-specific configuration factory at `0x10020715c` also sets preset ID `apple_dito_ph`, display name DITO, IPv4, IKE AES-128 / SHA-1 integrity / SHA-1 PRF / MODP-1024, and ESP AES-128 / SHA-1. It uses the standard 51566 ePDG and IMS realms, 20-second NAT keepalive, IPsec, 1300-byte UDP limit, P-CSCF attributes `[20, 21, 16384, 16386]`, and a 7200-second REGISTER expiry. The six singleton algorithm arrays and the four UInt16 attributes are independently decoded from static data; the expiry comes from its instruction and the reflected struct layout. The extraction manifest is `validation/dito-reference-v29.3.json`. `tests/check_dito_reference_package.py` verifies the IPA, exact function hash and these constants.

This Android patch separates operator recognition from the credentials needed for recovery. The earlier patch incorrectly restricted both to the reference's single `8963662600` batch. It now recognizes the user's known DITO IIN `896366` on 20404 roaming identities. An actually read, valid same-card 51566 identity also supplies stronger evidence across other ICCID issuers. Recognition never manufactures a home IMSI: recovery still needs the full, genuine same-ICCID home record learned at the generation-checked physical USIM read boundary.

The physical read result remains unchanged; a temporary identity is supplied to the existing native connection scheduler, so IKE/AKA and IMS receive consistent home credentials. Learning a home record invalidates an existing roaming configuration cache even if the raw card object remains the same. No SIM writes or cross-chip switch changes are introduced.

Automatic Wi-Fi configuration uses the dedicated reference DITO settings without depending on a picker lookup. The original native completion log used a local configuration from before the state constructor applied overrides, so it could still report Vodafone after the effective configuration changed; this local value is now resolved before logging. A fresh read also reports DITO recognition and whether a same-card home record exists, using the existing connection log.

Other cards retain native selection. Explicit carrier choices remain supported. The legacy saved key `51566|standard_plmn_51566` resolves to `51566|apple_dito_ph`; its selected indicator remains correct. DITO configuration records use a stable ICCID-based key; old current-IMSI or known-home-IMSI records are copied and validated without deleting the original. Existing SMSC and other edits survive the identity change. Existing identity hiding and session log redaction cover the remembered home IMSI.

The DITO preset now matches the independently inspected reference package, including the existing Android engine's REGISTER expiry support. This is reference-package provenance, not carrier confirmation or proof of successful registration. The existing editors can still override its fields. Automatic IPv4 and exact DITO negotiation lists are respected by the existing engine adapters.

The update keeps the current Android layout and SMSC save fix, uses the existing long-term certificate, and retains the native APDU, AKA and switch workers. Automated checks exercise identity isolation, persistence, recovery, invalid inputs, settings migration, SMSC save/reopen and native DEX hook types, including the completion log hook.

`tests/check_native_carrier_runtime.py` additionally executes the actual APK's parser, preset builder, helper code and native model constructors after DEX-to-JVM translation. Its explicit constructor-forwarding and interface-call adapters accommodate R8's DEX-only optimizations; Android platform/session objects are doubles. The prior shipped APK reproduces the other-batch Vodafone regression as a negative control. JVM probes do not execute Android ART or the phone's radio, OMAPI or network. No connected-phone or carrier-network test has been performed.

After installing, first read this exact DITO profile while its actual IMSI starts `51566`. The app remembers that full identity locally; subsequent `20404` reads can restore it for Wi-Fi Calling. Old builds and the iOS app do not automatically transfer this app-private record. Clearing app data or uninstalling removes it. If only `20404` is available and this app has never saved the real home identity, this update cannot reconstruct it.

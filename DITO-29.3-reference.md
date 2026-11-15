# DITO home identity recovery

Reference: https://github.com/maow318/simhubforum/releases/tag/v29.3

The reference repository publishes iOS binaries, not the implementation source. Its release notes explicitly require reading the card once while it has its real home 515-66 identity. The standard IPA's SHA-256 is `1d0b3c0bea93695bf467efda8c3b07793b03532fd55c096207177e8f2edf2b5d`.

Inspection of that unencrypted arm64 executable confirms:

- `DitoHomeIdentityStore` persists the full, actually read home IMSI by ICCID, under `vowifi.dito.home_imsi.`. The save is read back and checked.
- The strict DITO issuer prefix is `8963662600`; accepted ICCIDs have 19 or 20 ASCII digits. A home IMSI has 14 or 15 ASCII digits, begins `51566`, and has a two-digit MNC. The validator starts at `0x100206364`.
- On a `20404` roaming identity, the resolver restores only the same ICCID's saved home IMSI. It creates a temporary identity with MNC length 2, preserving the remaining card metadata. Without a valid home record, Wi-Fi Calling is rejected with an explicit home-identity-required message. Restore entry points start at `0x1001800fc` and `0x100206488`; the save routine starts at `0x10017fe90`.
- It does not derive an original home subscriber number by replacing an IMSI prefix. Its strings explicitly say that a roaming IMSI cannot supply the missing home identity. Airplane mode before removal is advice for deregistration, not a way to recover a missing identity.

This Android patch independently implements that narrow same-card memory and restore flow. It learns only at the fresh, generation-checked USIM read boundary. The physical read result remains unchanged; a temporary identity is supplied to the existing native connection scheduler, so IKE/AKA and IMS receive consistent home credentials. No SIM writes or cross-chip switch changes are introduced.

Automatic Wi-Fi configuration chooses DITO for this exact issuer/identity combination, while other cards retain native selection. Explicit carrier choices remain supported. DITO configuration records now use a stable ICCID-based key; old current-IMSI or known-home-IMSI records are copied and validated without deleting the original. Existing SMSC and other edits survive the identity change. Existing identity hiding and session log redaction also cover the remembered home IMSI.

The bundled DITO preset remains the previously generated, unverified standard 51566 template. The new identity flow does not make that template carrier-confirmed or prove successful network registration. The Android change does not copy unrelated iOS features, SIP quirks, UI or private signing material.

The update keeps the current Android layout and SMSC save fix, uses the existing long-term certificate, and retains the native APDU, AKA and switch workers. Automated checks exercise identity isolation, persistence, recovery, invalid inputs, settings migration, SMSC save/reopen and native DEX hook types. No connected-phone or carrier-network test has been performed in this environment.

After installing, first read this exact DITO profile while its actual IMSI starts `51566`. The app remembers that full identity locally; subsequent `20404` reads can restore it for Wi-Fi Calling. Old builds and the iOS app do not automatically transfer this app-private record. Clearing app data or uninstalling removes it. If only `20404` is available and this app has never saved the real home identity, this update cannot reconstruct it.

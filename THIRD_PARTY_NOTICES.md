# Third-party components

The original euer-invoice Python code is proprietary; this does not change the
licenses of third-party components distributed with the separate Java toolchain.
No additional Python runtime packages are installed by this package.

The toolchain's `legal/` directory contains its component inventory, original
license and attribution files, and corresponding source archives for the covered
Saxon and validation-rule components. These materials are available independently
of a paid euer-invoice key. The build maintains the source catalogs under
`licenses/third-party/` and verifies the distributed materials before packaging.

The toolchain includes Mustangproject and KoSIT (Apache-2.0 for their own code),
Saxon-HE (MPL-2.0), EN16931 rules (EUPL-1.2), XRechnung configurations and rules,
PDFBox and other Java dependencies under their individual licenses. The only
bundled font is PDFBox's unmodified Liberation Sans fallback (SIL OFL-1.1).
Unused GNU FreeSans and Adobe Source font resources are excluded from our engine.
UBL/CII schemas and other resources retain their own license/copyright notices.
The toolchain does not bundle a Java runtime or Maven.

The Ed25519 arithmetic uses the algorithms also illustrated by the public-domain
Ed25519 reference implementation credited to Daniel J. Bernstein and contributors:
https://ed25519.cr.yp.to/software.html . This attribution preserves the reference's
origin; the algorithm is described in RFC 8032:
https://www.rfc-editor.org/rfc/rfc8032.html . Local validation and license-payload
handling are part of euer-invoice. The maintainers' provenance review is recorded
in docs/licensing-risk.md.

Third-party licenses, including rights to modify and redistribute their components
and obtain their covered source code, take precedence over any conflicting terms
of the proprietary product license. A paid key controls euer-invoice issuance
features; it does not restrict those third-party rights.

# Cerebro Verifier

Dette offentlige repositoriet inneholder to eksisterende GitHub Actions-arbeidsflyter for avgrenset PowerShell-verifikasjon. De skriver kvitteringer som workflowartefakter. Repositoriet er verifikasjonsgrunnlag; det installerer ikke Cerebro og dokumenterer ikke en bestemt lokal Working Source eller ekstern Context-deployment.

Cerebros definisjoner eies av [Cerebro-Source-1.0](https://github.com/morgul-tech/Cerebro-Source-1.0). Source-versjonen som en kandidat skal prøves mot, velges eksplisitt som en full commit-SHA. En omtalt eller tidligere grønn kjøring er ikke bevis for en ny kandidat.

## Velg eksisterende arbeidsflyt

| Behov | Inngang | Forutsetninger | Kvittering og bevisgrense |
| --- | --- | --- | --- |
| Prøve PowerShell-kapasiteten på en GitHub-runner | [PowerShell Capability Proof](.github/workflows/capability-proof.yml), via `workflow_dispatch` | Autorisert dispatch og tilgjengelig `windows-2022`-runner. Ingen workflowinput er deklarert. | `capability-receipt.json`, artefaktnavn `powershell-capability-receipt`, `verifier=CEREBRO-PS-CAPABILITY-001`. Prøver parser positivt/negativt, utføring, exitkode, stdout/stderr og et midlertidig Git-repository på denne runneren. |
| Prøve en eksakt Level-3-PowerShell-kandidat mot en eksakt Source-basis | [Verify Exact Level 3](.github/workflows/verify-level3.yml), via `workflow_dispatch` | Autorisert dispatch, tre eksakte input nedenfor, og en kandidat som følger workflowens kall- og resultatkontrakt. | `exact-level3-receipt.json`, artefaktnavn `exact-level3-receipt`, `verifier=CEREBRO-EXACT-L3-006`. Binder kvitteringen til kandidatens SHA256, Source-basis og faktisk testmiljø. |

Verdiene i feltet `verifier` er kvitteringsidentifikatorer. Denne README-en oppretter ingen ny Tool Contract eller `tool_id`.

## Exact Level 3: input og faktisk prøve

Arbeidsflyten krever:

- `expected_sha256`: SHA256 med 64 heksadesimale tegn for de eksakte, ukomprimerte kandidatbytene.
- `expected_base`: full Source-commit med 40 heksadesimale tegn.
- `payload_gzip_base64url`: GZIP-komprimerte kandidatbyte kodet som BASE64URL uten padding.

Arbeidsflyten sjekker ut `morgul-tech/Cerebro-Source-1.0` ved `expected_base` i en isolert fixture, rekonstruerer kandidaten, kontrollerer transportens roundtrip og SHA256, og parser filen. Kandidaten kjøres deretter med Windows PowerShell 5.1 og parameterne `-RepoPath`, `-ExpectedBranch main` og `-ExpectedBase`.

Prøvene omfatter vellykket utføring, skittent arbeidsområde, feil branch/detached HEAD, feil baseline, injisert feil etter skriving med kontroll av rollback, og avvisning av dobbeltutføring. Suksesscenarioet krever JSON-resultat med `state=PASS` og et sannhetsverdi-resultat i `validation`. Det er workflowkoden som fastsetter den eksakte kontrakten.

## Bruk resultatet innen det prøvde omfanget

Les kvitteringens kandidat-SHA256, baseline, PowerShell-versjon og delresultater før `overall` brukes. Knytt resultatet til faktisk workflowrun og artefakt. En kapasitetstest verifiserer runnerens prøvde egenskaper; den verifiserer ikke automatisk en kandidat. En Level-3-kvittering gjelder den eksakte kandidaten og fixturen, og er ikke automatisk lokal innføring, Source-publisering, runtimekontroll eller produksjonsaksept.

Repoets eier er `morgul-tech`. Den ansvarlige for den konkrete kandidatleveransen eier input, valgt bevisbehov, mottak og videre disposisjon. Manglende dispatchadgang, kvittering eller nødvendig målmiljøbevis skal angis presist og føres tilbake dit. Lesing av disse filene krever ingen workflowkjøring.

Ved behov for å publisere Working Source, bruk Source-eide [synkdefinisjoner](https://github.com/morgul-tech/Cerebro-Source-1.0/blob/e53dfde705f59b24ad4f55f14260670bc323c744/modules/terminology/terms.yaml) og [Source Authority Standard](https://github.com/morgul-tech/Cerebro-Source-1.0/blob/e53dfde705f59b24ad4f55f14260670bc323c744/standards/source-authority.yaml). Verifier-kvitteringer erstatter ikke kanonisk `cerebro_sync` med fersk uavhengig kontroll av likhet mellom Working Source og autoritativ remote. Disse Source-pekerne er dokumentasjonsgrunnlag kontrollert 18. oktober 2026.


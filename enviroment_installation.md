# Prerequisiti ambiente di sviluppo e test

Questo documento definisce cosa deve essere installato sulla macchina prima di iniziare sviluppo, test o debug dell'applicazione.

## 1) Componenti obbligatori

### Backend (.NET)
- .NET runtime target: `.NET 8` LTS.
- .NET SDK richiesto: `8.0.424`.

Comando di verifica:

```powershell
dotnet --list-sdks
```

Esito atteso: presenza di `8.0.424`.

### Frontend toolchain
- `Node.js 22.18.0` LTS.
- `npm 10.9.3`.

Comandi di verifica:

```powershell
node -v
npm -v
```

Esito atteso:
- `node -v` -> `v22.18.0`
- `npm -v` -> `10.9.3`

### Browser supportati (esecuzione app)
- Versione stabile di almeno uno tra: `Chrome`, `Edge`, `Firefox`.
- Supporto `WebGL 2.0` attivo.

Verifica rapida:
- aprire il browser target e controllare che WebGL 2.0 sia disponibile.

## 2) Dipendenze applicative baseline (vincolanti)

Le versioni baseline sono definite dalle specifiche e devono essere rispettate in sviluppo/CI:

- Frontend runtime: `three@0.161.0`, `@dimforge/rapier3d-compat@0.12.0`.
- Frontend tooling: `vite@5.2.0`, `typescript@5.4.5`.
- Frontend test: `vitest@1.6.0`, `@playwright/test@1.44.0`.
- Backend runtime: `Serilog.AspNetCore@8.0.1`, `Serilog.Sinks.File@5.0.0`, `SharpGLTF.Core@1.0.0`, `MIConvexHull@1.1.19.504`.
- Backend test: `xunit@2.7.1`, `Microsoft.AspNetCore.Mvc.Testing@8.0.5`.

Regole obbligatorie:
- vietate versioni floating (`^`, `~`, wildcard);
- vietate dipendenze prerelease (`-alpha`, `-beta`, `-rc`);
- lockfile npm/NuGet obbligatori e versionati;
- versioni NuGet dirette espresse come intervalli esatti (esempio: `[1.1.19.504]`).

Nota operativa: queste librerie non vanno installate globalmente sulla macchina; vengono ripristinate dal progetto tramite lockfile.

## 3) Checklist pre-avvio (go/no-go)

Eseguire nell'ordine:

1. `dotnet --list-sdks` e verificare `8.0.424`.
2. `node -v` e verificare `v22.18.0`.
3. `npm -v` e verificare `10.9.3`.
4. Verificare disponibilita browser stabile con WebGL 2.0.
5. Verificare che il progetto includa lockfile npm e NuGet prima del restore/build.

Se uno dei punti non e soddisfatto, fermare attivita di sviluppo/test e allineare l'ambiente.

## 4) Riferimenti specifiche

- `sdd-specs/01-specifica-requisiti.md` (REQ-DEP-001..004, REQ-ARC-007, REQ-TST-002, REQ-TST-005, REQ-TST-006)
- `sdd-specs/02-design-tecnico.md` (sezione 12)
- `sdd-specs/00-costituzione.md` (C12, C13, C15)

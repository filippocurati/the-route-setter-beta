# 09 — Requirements Traceability

## 1. Scopo

Questa matrice collega i requisiti di prodotto alle specifiche SDD e ai test.

| Requisito | Specifica SDD | Test principale |
|---|---|---|
| FR-001 | 02, 03, 05 | E2E startup |
| FR-002 | 02, 05, 06 | E2E catalog |
| FR-003 | 02, 05 | E2E/cache behavior |
| FR-004 | 02, 03, 04, 05 | E2E utilizzo |
| FR-005 | 02, 04, 05 | E2E rimozione + PHY-004 |
| FR-006 | 02, 05 | E2E selezione |
| FR-007 | 02, 04, 05 | input/E2E |
| FR-008 | 02, 04, 05 | input/E2E |
| FR-009 | 02, 04 | geometry/interaction tests |
| FR-010 | 02, 05 | input/E2E |
| FR-011 | 02, 04 | TEST-SNAP-001/002 |
| FR-012 | 02, 04 | TEST-SNAP-003 |
| FR-013 | 02, 04 | TEST-SNAP-004 |
| FR-014 | 02, 04 | TEST-SNAP-005 |
| FR-015 | 02, 04 | TEST-PHYS-001/002/005 |
| FR-016 | 02, 04 | physics suite |
| FR-017 | 02, 05, 06 | asset tests |
| FR-018 | 02, 05 | E2E Details |
| FR-019 | 02, 04, 06 | hull tests |
| FR-020 | 02, 06 | TEST-HULL-001 |
| FR-021 | 02, 06 | TEST-HULL-002/003 |
| FR-022 | 02, 06 | integration/background tests |
| FR-023 | 02, 04, 05 | physics/E2E |
| FR-024 | 05 | E2E camera |
| FR-025 | 02, 05 | E2E image generation |
| FR-026 | 01, 02 | architecture review |
| FR-027 | 07 | error tests |
| FR-028 | 07 | logging tests |
| FR-029 | 01, 05, 08 | documentation verification |
| PHY-001..009 | 04 | physics suite |
| NFR-001 | 01, 05 | browser compatibility |
| NFR-002 | 01, 05 | performance validation |
| NFR-003 | 01, 05 | 40-hold test |
| NFR-004 | 01, 05 | interaction validation |
| NFR-005 | 01, 05, 08 | performance validation |
| NFR-006 | 05 | loading E2E |
| NFR-007 | 08 | CI |
| NFR-008 | 04, 08 | deterministic physics |
| NFR-009 | 07 | log sanitization |
| NFR-010 | 07 | log rotation |

## 2. Traceability rule

Ogni nuova funzionalità implementata dall'agente deve essere associata:
1. a uno o più requisiti;
2. a uno o più criteri di accettazione;
3. a un test quando il requisito è automaticamente verificabile.

Una funzionalità non richiesta non deve essere introdotta solo perché tecnicamente comoda.

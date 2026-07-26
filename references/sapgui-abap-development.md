# SAP GUI ABAP Development Notes

Use this reference before changing ABAP through SAP GUI, especially SE38 or SE11.

## SAP Editor Write Safety

- Do not trust clipboard-based paste as proof that SAP was updated.
- Treat these as separate states: local file updated, SAP editor buffer updated, SAP object saved, SAP object syntax-checked, SAP object activated.
- Keep the SAP repository object name, local ABAP `REPORT` statement, local file name, and sync script default target aligned. In this project the active report object is `ZPP_ROUTING_BOM_UPLOAD`; writing to the older `ZPP_ROUTING_UPLOAD` target fails because that object is not the live SAP report.
- If the user says SAP did not update, believe the user's screen and re-check from SAP, not from the local clipboard.
- Prefer SAP editor upload from local file when available: `Utilities -> More Utilities -> Upload/Download -> Upload...`.
- On old SAP GUI/ECC systems, SAP editor upload may read local files using the frontend/SAP code page rather than UTF-8. If the ABAP source contains Chinese or other non-ASCII string literals, uploading a UTF-8 file can corrupt those literals in SAP. Prefer ASCII literals for sync-safe ABAP source, or explicitly prepare a file in the frontend code page before upload.
- If using paste, only claim success after a SAP-side check that does not reuse the same local clipboard contents.
- A SAP editor title containing `Display` means writes may be blocked. In Chinese SAP GUI this is the display-mode report editor title. Re-open from SE38 initial screen with the program name and the `Change` button.
- If SAP reports another user or the same user is editing the object, stop and resolve the lock/mode before editing.
- Do not use activation as a substitute for syntax check when the user asked for syntax check. Press the SE38 `Check` button and read the status bar.

## Verification Pattern

For SAP GUI ABAP changes, verify in this order:

1. Save status bar shows the program was saved.
2. SE38 syntax-check button returns `Program <name> is syntactically correct`.
3. Activation status bar shows the object was activated, when activation is needed.
4. If SAP write reliability is in question, read the SAP source back and compare hashes or a targeted diff against the local source. Do not rely on copying back from the Windows clipboard if that clipboard was just used for paste.

## Known Compatibility Lessons

- The target SAP system may be old ECC 6.0 / ABAP 7.30. Prefer conservative classic ABAP syntax unless the exact release supports newer syntax.
- Avoid ABAP 7.40+ style by default: inline declarations like `DATA(...)`, constructor expressions like `VALUE #( )`, table expressions like `itab[ key ]`, `@` Open SQL host variables, `NEW`, `CONV`, `COND`, `SWITCH`, and other modern shorthand.
- Prefer explicit declarations and classic forms: `DATA lv_x TYPE ...`, `READ TABLE ... INTO ... WITH KEY ...`, `CALL METHOD ... EXPORTING ...`, `CALL FUNCTION ...`, `LOOP AT ... INTO ...`, and classic Open SQL without `@`.
- `CL_SALV_FUNCTIONS_LIST->ADD_FUNCTION`: do not pass `ICON_SYSTEM_SAVE` blindly. Some systems report `ICON_SYSTEM_SAVE is not type-compatible with formal parameter ICON`. Use text/tooltip only unless the target system accepts the icon type.
- On old ECC systems, SALV function enablement can dump with `CX_SALV_METHOD_NOT_SUPPORTED` and `Only Possible in Grid View`. When using SALV custom functions, force grid display with `CL_SALV_TABLE=>FACTORY EXPORTING list_display = space` and catch `CX_SALV_METHOD_NOT_SUPPORTED`.
- If `CL_SALV_FUNCTIONS_LIST->ADD_FUNCTION` still raises `CX_SALV_METHOD_NOT_SUPPORTED` / "ALV grid function is not supported", create the SALV in a GUI container, for example `CL_GUI_CONTAINER=>DEFAULT_SCREEN`, instead of fullscreen SALV without a container. Then call `DISPLAY` and output a harmless `WRITE space` so the default screen is rendered.
- Avoid `lo_functions->set_all( abap_true )` when adding a single custom SALV button on old ECC unless the standard functions have been proven compatible. It can enable unsupported standard functions before display.
- Do not use standard-looking SALV function codes like `SAVE` for custom actions. Use a customer namespace code such as `ZSAVE` and check that exact code in `on_added_function`.
- SALV custom buttons require both `ADD_FUNCTION` and event registration:
  - Create/get `CL_SALV_EVENTS_TABLE`.
  - Create the event handler object.
  - `SET HANDLER handler->on_added_function FOR events`.
- When reusing standard SALV/PF-STATUS buttons instead of adding a custom button, register `AFTER_SALV_FUNCTION` as well as any custom-function handler. Standard toolbar functions are not guaranteed to raise `ADDED_FUNCTION`.
- Be careful reusing SALV/PF-STATUS standard save buttons such as `&DATA_SAVE` for business imports. In ECC SALV they can be disabled/grey if SAP treats them as layout or variant save functions, not an application save action. If the user explicitly requires the `STANDARD` status save button, use `SET_SCREEN_STATUS` with that PF-STATUS and handle `AFTER_SALV_FUNCTION`; if it is still grey, inspect the GUI status definition in SE41 because code cannot trigger a disabled toolbar function.
- In SALV fullscreen, a `STANDARD` PF-STATUS button may not reach report-level `AT USER-COMMAND`; SALV owns the screen flow. If a standard-status button is clickable but `AFTER_SALV_FUNCTION` does not fire, also register `ADDED_FUNCTION` and route both event function codes through the same handler. Common save codes to cover include `&DATA_SAVE`, `&SAVE`, `SAVE`, and `SICH`; if still not triggered, debug the event handler input to identify the exact function code maintained in SE41.
- If import should happen from ALV, do not call save logic after `go_alv->display( )` returns. Put save logic in the ALV function event and refresh ALV after results are written.
- For per-row or per-group result display, write `status`, `message`, and saved identifiers back into the ALV internal table before `go_alv->refresh( )`.
- Keep count constants aligned with the visible structures. If an ALV/output type has fields such as `LSTAR1` through `LSTAR6`, the loop limit and error message must also allow 6 entries; otherwise the final visible slot is never filled.
- On old ECC systems, prefer `DESCRIBE TABLE ... LINES ...` for diagnostic row counts before ALV display. This gives a clear program-controlled error when preview data is empty instead of relying on generic SAP list messages such as "no data selected".
- When a custom table controls routing activity-type slots, clarify whether the sequence is compact ordering or a direct BAPI slot number. In this project `ZPP_WC_ACT_STD-SORTNO` is a direct slot: `01` fills `ACTTYPE_01`, `05` fills `ACTTYPE_05`, and missing slots stay blank.
- Avoid `AT NEW <field>` for grouping when the line type's field order does not exactly match the intended grouping key. On old ABAP, control-break processing depends on the structure field sequence; use explicit previous-key variables for groups like `MATNR + WERKS`.
- For routing creation from material input, avoid hardcoded task or operation units such as `KG` or `PC`. Read `MARA-MEINS` for `TASK_MEASURE_UNIT` and `OPERATION_MEASURE_UNIT`, and read `MAKT-MAKTX` for task description when the requirement says to use the material description.
- For routing operation descriptions, do not blindly use the work center code `ARBPL` if the requirement says to use the work center description. Read `CRHD` by plant/work center to get `OBJTY/OBJID`, then read `CRTX-KTEXT` by object and `SPRAS = SY-LANGU`; fall back to `ARBPL` only when no text is maintained.
- For ALV result indicators on old ECC, keep a separate business status field for logic and add a first display field typed `ICON_D` for red/yellow/green lights. Update the icon whenever status or saved state changes, then hide the internal status column if the user only needs the light.

## BAPI and DDIC Lessons

- `BAPI_ROUTING_CREATE` `TESTRUN` may have a system-specific type. Passing `space` can dump with `CALL_FUNCTION_CONFLICT_TYPE` / `CX_SY_DYN_CALL_ILLEGAL_TYPE`. Omit optional `TESTRUN` unless the interface type has been checked or use a correctly typed variable.
- `CSAP_MAT_BOM_CREATE` `VALID_FROM` can dump with `CX_SY_DYN_CALL_ILLEGAL_TYPE` if called with `sy-datum` directly on old ECC systems. Declare a variable with the exact interface type, e.g. `DATA lv_valid_from TYPE csap_mbom-datuv`, fill it via `WRITE sy-datum TO lv_valid_from`, and pass that variable to `VALID_FROM`.
- Quantity fields added to DDIC tables need reference table/field. A `QUAN` field without a unit reference causes table check errors.
- When a standard-value table stores base quantity and uploaded data has operation base quantity, clarify whether base quantity is part of lookup or a conversion factor. In this project the lookup is by `WERKS + ARBPL`; standard values are scaled by `uploaded_bmsch / table_bmsch`.
- For Excel uploads, confirm the template's first data row before changing `ALSM_EXCEL_TO_INTERNAL_TABLE` `i_begin_row`. If the user says data starts on row 2, keep `i_begin_row = 2`; do not "fix" it to row 1. Add `error_message` to the function module exceptions and check `lt_excel IS INITIAL` so standard "no data" messages become clear diagnostics.
- For selection-screen template download buttons, do not make the upload file parameter `OBLIGATORY` if the user should be able to download a template before choosing a file. Add `SELECTION-SCREEN FUNCTION KEY`, set `SSCRFIELDS-FUNCTXT_01` in `INITIALIZATION`, handle `SSCRFIELDS-UCOMM = 'FC01'` in `AT SELECTION-SCREEN`, and validate the file path later in `START-OF-SELECTION`.

## Red Flags

- "SAP is updated" based only on local git diff or local file contents.
- "SAP is updated" based on clipboard contents after a paste operation.
- Saving from a screen titled `Display`.
- Activating before reading SE38 syntax-check output.
- Adding SALV icons/constants without checking the target SAP release.
- Passing literal `space` to optional BAPI parameters without checking parameter type.

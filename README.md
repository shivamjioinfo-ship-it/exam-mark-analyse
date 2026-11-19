<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="Local-first exam marks support-ticket triage and ERP staging tool.">
  <title>Exam Marks Triage Desk</title>
  <style>
    :root {
      color-scheme: light;
      --ink: #17243b;
      --muted: #64748b;
      --line: #dce4ef;
      --paper: #fff;
      --canvas: #f3f6fb;
      --blue: #175cd3;
      --blue-pale: #eef4ff;
      --green: #067647;
      --green-pale: #ecfdf3;
      --amber: #92400e;
      --amber-pale: #fffaeb;
      --red: #b42318;
      --red-pale: #fef3f2;
      font: 15px/1.5 "Segoe UI", Arial, sans-serif;
    }
    * { box-sizing: border-box; }
    body { margin: 0; color: var(--ink); background: var(--canvas); }
    header { background: #102a56; color: white; padding: 22px max(24px, calc((100% - 1160px) / 2)); }
    header h1 { margin: 0; font-size: 1.45rem; }
    header p { margin: 4px 0 0; color: #cfdbef; }
    main { max-width: 1160px; margin: 24px auto 60px; padding: 0 20px; }
    .notice { border: 1px solid #f3cf86; background: var(--amber-pale); color: #713b12; border-radius: 10px; padding: 13px 16px; margin-bottom: 18px; }
    .notice strong { display: block; }
    .cards { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 12px; margin: 16px 0; }
    .card, section { background: var(--paper); border: 1px solid var(--line); border-radius: 12px; }
    .card { padding: 15px 17px; }
    .card span { display: block; color: var(--muted); font-size: .88rem; }
    .card strong { display: block; margin-top: 5px; font-size: 1.55rem; }
    section { padding: 20px; margin-top: 16px; }
    h2 { font-size: 1.13rem; margin: 0 0 12px; }
    h3 { font-size: 1rem; margin: 18px 0 8px; }
    .subtle { color: var(--muted); font-size: .9rem; }
    .uploads { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
    label { display: block; font-weight: 600; margin-bottom: 6px; }
    input[type=file] { display: block; width: 100%; padding: 11px; border: 1px dashed #9aabc2; border-radius: 8px; background: #fbfcff; }
    .actions { display: flex; flex-wrap: wrap; gap: 9px; margin-top: 14px; align-items: center; }
    button, .button { border: 0; border-radius: 7px; padding: 10px 14px; font: inherit; font-weight: 650; cursor: pointer; }
    button.primary { color: white; background: var(--blue); }
    button.secondary { color: #243b60; background: #e9eef6; }
    button:disabled { opacity: .48; cursor: not-allowed; }
    button:focus-visible, input:focus-visible { outline: 3px solid #84adff; outline-offset: 2px; }
    .status { min-height: 1.5em; color: var(--muted); }
    .table-wrap { overflow-x: auto; }
    table { border-collapse: collapse; width: 100%; font-size: .9rem; }
    th, td { padding: 10px 9px; text-align: left; vertical-align: top; border-bottom: 1px solid var(--line); }
    th { color: #34445d; background: #f8faff; white-space: nowrap; }
    td input[type=checkbox] { width: 18px; height: 18px; accent-color: var(--blue); }
    .pill { display: inline-block; border-radius: 99px; padding: 2px 9px; font-size: .78rem; font-weight: 700; white-space: nowrap; }
    .pill.ready { color: var(--green); background: var(--green-pale); }
    .pill.review { color: var(--amber); background: var(--amber-pale); }
    .pill.blocked { color: var(--red); background: var(--red-pale); }
    .empty { color: var(--muted); padding: 15px 4px; }
    .ticket-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
    .ticket { border: 1px solid var(--line); border-radius: 9px; padding: 14px; background: #fbfcff; }
    .ticket h3 { margin-top: 0; }
    .ticket p { margin: 7px 0; }
    pre { overflow-x: auto; padding: 13px; border-radius: 8px; background: #101d32; color: #e3edff; font-size: .84rem; }
    footer { color: var(--muted); text-align: center; padding: 22px; font-size: .85rem; }
    @media (max-width: 760px) {
      .cards { grid-template-columns: repeat(2, minmax(0, 1fr)); }
      .uploads, .ticket-grid { grid-template-columns: 1fr; }
      header { padding: 20px; }
    }
  </style>
</head>
<body>
  <header>
    <h1>Exam Marks Triage Desk</h1>
    <p>Two-ticket workflow · anomaly checks · review-gated ERP staging</p>
  </header>
  <main>
    <div class="notice" role="note">
      <strong>Demo only — no real ticket or exam data was supplied.</strong>
      Sample records are synthetic. Files stay in this browser session; this generic CSV is not an ERP-approved import format.
      Verify against the signed source record, confirm the applicable scheme, and use an ERP dry run before any real upload.
    </div>

    <div class="cards" aria-label="Triage summary">
      <div class="card"><span>Records loaded</span><strong id="count-total">0</strong></div>
      <div class="card"><span>Missing report marks</span><strong id="count-missing">0</strong></div>
      <div class="card"><span>Missing / unknown scheme</span><strong id="count-scheme">0</strong></div>
      <div class="card"><span>Rows eligible for review</span><strong id="count-ready">0</strong></div>
    </div>

    <section>
      <h2>1. Load and validate records</h2>
      <p class="subtle">Marks CSV columns: <code>ticket_id,student_id,exam_code,course_code,source_marks,report_marks,scheme_code</code>. Scheme CSV: <code>scheme_code,course_code,max_marks</code>. Student identifiers are shown only in this local browser tab.</p>
      <div class="uploads">
        <div><label for="marks-file">Exam report / source marks CSV</label><input id="marks-file" type="file" accept=".csv,text/csv"></div>
        <div><label for="schemes-file">Approved schemes CSV</label><input id="schemes-file" type="file" accept=".csv,text/csv"></div>
      </div>
      <div class="actions">
        <button id="analyze" class="primary" type="button">Analyze uploaded files</button>
        <button id="demo" class="secondary" type="button">Load synthetic demo</button>
        <button id="clear" class="secondary" type="button">Clear data</button>
      </div>
      <p id="status" class="status" role="status" aria-live="polite"></p>
    </section>

    <section>
      <h2>2. Review findings and verify candidate marks</h2>
      <p class="subtle">A checkbox is available only when the row has a known scheme, valid source marks, and no duplicate student/exam/course key. Check it only after independent source-record verification. Blocked rows are never exported.</p>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Ticket</th><th>Student</th><th>Exam / course</th><th>Source</th><th>Reported</th><th>Scheme</th><th>Finding / status</th><th>Verified</th></tr></thead>
          <tbody id="results"><tr><td class="empty" colspan="8">Upload CSV files or load the synthetic demo to begin.</td></tr></tbody>
        </table>
      </div>
      <div class="actions">
        <button id="export" class="primary" type="button" disabled>Download reviewed staging CSV</button>
        <span class="subtle">Only individually verified, anomaly-free candidate rows are included.</span>
      </div>
    </section>

    <section>
      <h2>3. Ticket close-out drafts</h2>
      <p class="subtle">Use these only after replacing placeholders with verified findings and obtaining the required academic approval.</p>
      <div class="ticket-grid">
        <article class="ticket">
          <h3>Client / student-facing update</h3>
          <p><strong>Subject:</strong> Update on exam marks query [ticket ID]</p>
          <p>Hello [name], we have logged your report for [exam/course]. We are checking the published result against the approved source marks and applicable scheme. We will update you by [time within the agreed service window]. No mark has been changed at this stage. If a correction is confirmed and approved, we will notify you after the authorized result update. Please do not send sensitive student information by unencrypted email.</p>
          <p><strong>Close only when:</strong> the verified outcome and approval reference have been shared, or the requester confirms no further help is needed. Otherwise keep the ticket open with the next update time.</p>
        </article>
        <article class="ticket">
          <h3>Developer / exam-office handoff</h3>
          <p><strong>Subject:</strong> Investigate missing marks — [ticket ID, exam, course]</p>
          <p>Observed: [missing / mismatched marks and affected record count]. Checks: [source reconciliation, duplicate-key check, range check, scheme lookup]. Evidence: [restricted system reference, query/job ID, timestamp; no student PII in email]. Please confirm the result publication/import job, scheme mapping and any rejected-row logs. Proposed action: [correct rejected rows / restore missing scheme / no change], pending exam-office approval. Validate counts and a sample record in test or dry run before production; retain rollback and audit references.</p>
          <p><strong>Close only when:</strong> root cause, approved correction/no-change decision, reconciliation counts and post-change verification are recorded.</p>
        </article>
      </div>
    </section>

    <section>
      <h2>Workflow evidence and verification checklist</h2>
      <h3>AI-assisted example (synthetic records only)</h3>
      <p><strong>Bounded prompt:</strong> “Classify these anonymized mark-report anomalies, suggest read-only checks, and draft neutral updates. Do not infer or approve marks. Flag missing schemes, duplicate keys, and out-of-range values.”</p>
      <p><strong>Example triage:</strong> DEMO-001 has no report mark; source is 82 and the scheme maximum is 100. DEMO-002 has a source/report mismatch (76 vs 67). A row with SCHEME-MISSING is blocked. These are synthetic inputs; the app's deterministic checks—not AI—produce the displayed findings. No candidate becomes exportable until a human checks source verification, and approval plus ERP mapping remain mandatory.</p>
      <ol>
        <li>Use AI for issue classification, read-only SQL drafts, and neutral communication templates; provide anonymized/minimized data only and verify every result against the source system.</li>
        <li>Use registrar-approved source data and schemes. Reconcile every candidate to the original signed/approved record and record who verified it.</li>
        <li>Check duplicates, missing values, scheme coverage, and mark ranges; resolve every blocker before staging.</li>
        <li>Compare CSV row counts and a sample to the approved correction list. Use the vendor/ERP template, access controls, dry run, approval, audit trail and rollback process.</li>
        <li>After upload, query the ERP again, reconcile accepted/rejected counts, verify the affected results, and update each ticket with factual evidence.</li>
      </ol>
      <p class="subtle">Timebox: target triage and initial update within three hours; do not promise a marks correction within that period unless approval and system verification are complete.</p>
    </section>
  </main>
  <footer>Local-first prototype · Does not connect to ERP or transmit uploaded files</footer>

  <script>
    "use strict";
    const state = { records: [], schemes: [] };
    const requiredMarks = ["ticket_id", "student_id", "exam_code", "course_code", "source_marks", "report_marks", "scheme_code"];
    const requiredSchemes = ["scheme_code", "course_code", "max_marks"];
    const $ = (id) => document.getElementById(id);
    const escapeHtml = (value) => String(value ?? "").replace(/[&<>"']/g, (char) => ({
      "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;"
    })[char]);

    function parseCsv(text) {
      const rows = [];
      let row = [], field = "", quoted = false;
      const input = text.replace(/^\uFEFF/, "");
      for (let i = 0; i < input.length; i++) {
        const char = input[i];
        if (quoted) {
          if (char === '"' && input[i + 1] === '"') { field += '"'; i++; }
          else if (char === '"') quoted = false;
          else field += char;
        } else if (char === '"' && field === "") quoted = true;
        else if (char === ",") { row.push(field); field = ""; }
        else if (char === "\n" || char === "\r") {
          if (char === "\r" && input[i + 1] === "\n") i++;
          row.push(field); field = "";
          if (row.some((cell) => cell.trim() !== "")) rows.push(row);
          row = [];
        } else field += char;
      }
      if (quoted) throw new Error("CSV contains an unclosed quoted field.");
      row.push(field);
      if (row.some((cell) => cell.trim() !== "")) rows.push(row);
      if (rows.length < 2) throw new Error("CSV must include a header and at least one data row.");
      const headers = rows[0].map((header) => header.trim().toLowerCase());
      if (new Set(headers).size !== headers.length) throw new Error("CSV contains duplicate column names.");
      return rows.slice(1).map((cells, index) => {
        if (cells.length !== headers.length) throw new Error(`CSV row ${index + 2} has ${cells.length} cells; expected ${headers.length}.`);
        return Object.fromEntries(headers.map((header, i) => [header, cells[i].trim()]));
      });
    }

    function validateHeaders(rows, required, fileDescription) {
      const headers = new Set(Object.keys(rows[0] || {}));
      const missing = required.filter((key) => !headers.has(key));
      if (missing.length) throw new Error(`${fileDescription} is missing required columns: ${missing.join(", ")}.`);
    }

    function numberOrNull(value) {
      if (value === "") return null;
      const number = Number(value);
      return Number.isFinite(number) ? number : NaN;
    }

    function analyze() {
      const schemeMap = new Map();
      const duplicateSchemeKeys = new Set();
      state.schemes.forEach((scheme) => {
        const key = `${scheme.scheme_code}\u0000${scheme.course_code}`;
        if (schemeMap.has(key)) duplicateSchemeKeys.add(key);
        else schemeMap.set(key, numberOrNull(scheme.max_marks));
      });
      const rowKeys = state.records.map((record) =>
        `${record.student_id}\u0000${record.exam_code}\u0000${record.course_code}`
      );
      const counts = new Map();
      rowKeys.forEach((key) => counts.set(key, (counts.get(key) || 0) + 1));
      return state.records.map((record, index) => {
        const schemeKey = `${record.scheme_code}\u0000${record.course_code}`;
        const max = schemeMap.get(schemeKey);
        const source = numberOrNull(record.source_marks);
        const reported = numberOrNull(record.report_marks);
        const findings = [];
        let blocked = false;
        if (record.report_marks === "") findings.push("Missing report marks");
        else if (Number.isNaN(reported)) { findings.push("Invalid report marks"); blocked = true; }
        else if (!Number.isNaN(source) && source !== null && source !== reported) findings.push("Source/report mismatch");
        if (record.source_marks === "" || Number.isNaN(source)) { findings.push("Missing or invalid source marks"); blocked = true; }
        if (max === undefined || max === null || Number.isNaN(max) || max < 0 || duplicateSchemeKeys.has(schemeKey)) {
          findings.push(duplicateSchemeKeys.has(schemeKey) ? "Duplicate scheme entry" : "Missing/invalid scheme");
          blocked = true;
        }
        if (!blocked && (source < 0 || source > max)) { findings.push("Source marks out of range"); blocked = true; }
        if (counts.get(rowKeys[index]) > 1) { findings.push("Duplicate student/exam/course key"); blocked = true; }
        if (record.scheme_code === "" || record.student_id === "" || record.exam_code === "" || record.course_code === "" || record.ticket_id === "") {
          findings.push("Missing required identifier");
          blocked = true;
        }
        return { record, max, source, reported, findings, blocked, verified: Boolean(record._verified) };
      });
    }

    function render() {
      const analyzed = analyze();
      const missing = analyzed.filter((item) => item.record.report_marks === "").length;
      const schemesMissing = analyzed.filter((item) => item.findings.some((finding) => /scheme/i.test(finding))).length;
      const ready = analyzed.filter((item) => !item.blocked && item.findings.length > 0).length;
      $("count-total").textContent = String(state.records.length);
      $("count-missing").textContent = String(missing);
      $("count-scheme").textContent = String(schemesMissing);
      $("count-ready").textContent = String(ready);
      const tbody = $("results");
      if (!analyzed.length) {
        tbody.innerHTML = '<tr><td class="empty" colspan="8">Upload CSV files or load the synthetic demo to begin.</td></tr>';
      } else {
        tbody.innerHTML = analyzed.map((item, index) => {
          const needsAction = item.findings.length > 0;
          const pill = item.blocked ? "blocked" : needsAction ? "review" : "ready";
          const stateText = item.blocked ? "Blocked" : needsAction ? "Review candidate" : "No discrepancy";
          const verification = !item.blocked && needsAction
            ? `<label aria-label="Mark row ${index + 1} source-verified"><input type="checkbox" data-verify="${index}" ${item.verified ? "checked" : ""}> Source verified</label>`
            : "—";
          return `<tr>
            <td>${escapeHtml(item.record.ticket_id)}</td>
            <td>${escapeHtml(item.record.student_id)}</td>
            <td>${escapeHtml(item.record.exam_code)} / ${escapeHtml(item.record.course_code)}</td>
            <td>${escapeHtml(item.record.source_marks || "—")}${Number.isFinite(item.max) ? ` / ${escapeHtml(item.max)}` : ""}</td>
            <td>${escapeHtml(item.record.report_marks || "—")}</td>
            <td>${escapeHtml(item.record.scheme_code || "—")}</td>
            <td><span class="pill ${pill}">${stateText}</span>${item.findings.length ? `<br>${item.findings.map(escapeHtml).join("; ")}` : ""}</td>
            <td>${verification}</td>
          </tr>`;
        }).join("");
      }
      tbody.querySelectorAll("[data-verify]").forEach((checkbox) => {
        checkbox.addEventListener("change", (event) => {
          const row = Number(event.currentTarget.dataset.verify);
          const current = analyze();
          if (!current[row] || current[row].blocked) return;
          current[row].verified = event.currentTarget.checked;
          state.records[row]._verified = event.currentTarget.checked;
          render();
        });
      });
      $("export").disabled = !analyzed.some((item) => !item.blocked && item.findings.length > 0 && item.verified);
    }

    async function readCsv(fileInput, required, description) {
      const file = fileInput.files && fileInput.files[0];
      if (!file) throw new Error(`Select the ${description} CSV file.`);
      const rows = parseCsv(await file.text());
      validateHeaders(rows, required, description);
      return rows;
    }

    $("analyze").addEventListener("click", async () => {
      try {
        const [records, schemes] = await Promise.all([
          readCsv($("marks-file"), requiredMarks, "marks"),
          readCsv($("schemes-file"), requiredSchemes, "schemes")
        ]);
        state.records = records;
        state.schemes = schemes;
        render();
        $("status").textContent = `Analyzed ${records.length} marks rows against ${schemes.length} scheme rows.`;
      } catch (error) {
        $("status").textContent = error.message;
      }
    });

    $("demo").addEventListener("click", () => {
      state.schemes = [
        { scheme_code: "SCHEME-2026-A", course_code: "BIO101", max_marks: "100" },
        { scheme_code: "SCHEME-2026-A", course_code: "CHEM101", max_marks: "100" }
      ];
      state.records = [
        { ticket_id: "DEMO-001", student_id: "SYN-1001", exam_code: "EXAM-2026", course_code: "BIO101", source_marks: "82", report_marks: "", scheme_code: "SCHEME-2026-A" },
        { ticket_id: "DEMO-002", student_id: "SYN-1002", exam_code: "EXAM-2026", course_code: "CHEM101", source_marks: "76", report_marks: "67", scheme_code: "SCHEME-2026-A" },
        { ticket_id: "DEMO-002", student_id: "SYN-1003", exam_code: "EXAM-2026", course_code: "CHEM101", source_marks: "71", report_marks: "", scheme_code: "SCHEME-MISSING" }
      ];
      render();
      $("status").textContent = "Loaded synthetic demonstration rows. None are real student records.";
    });

    $("clear").addEventListener("click", () => {
      state.records = [];
      state.schemes = [];
      $("marks-file").value = "";
      $("schemes-file").value = "";
      $("status").textContent = "Cleared in-memory records.";
      render();
    });

    function csvCell(value) {
      const text = String(value ?? "");
      return `"${text.replace(/"/g, '""')}"`;
    }

    $("export").addEventListener("click", () => {
      const rows = analyze().filter((item) => !item.blocked && item.findings.length > 0 && item.record._verified);
      if (!rows.length) {
        $("status").textContent = "No verified candidate rows to export.";
        return;
      }
      const headers = ["ticket_id", "student_id", "exam_code", "course_code", "candidate_marks", "scheme_code", "source_marks", "previous_report_marks", "verification_status"];
      const lines = [headers, ...rows.map((item) => [
        item.record.ticket_id, item.record.student_id, item.record.exam_code, item.record.course_code,
        item.source, item.record.scheme_code, item.source, item.record.report_marks, "SOURCE_VERIFIED_REQUIRES_APPROVAL"
      ])].map((row) => row.map(csvCell).join(","));
      const blob = new Blob(["\uFEFF" + lines.join("\r\n")], { type: "text/csv;charset=utf-8" });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.href = url;
      link.download = "re-evaluation-staging-NOT-ERP-READY.csv";
      document.body.append(link);
      link.click();
      link.remove();
      URL.revokeObjectURL(url);
      $("status").textContent = `Exported ${rows.length} reviewed candidate row(s). Approval and ERP-template mapping are still required.`;
    });

    render();
  </script>
</body>
</html>


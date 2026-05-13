import tkinter as tk
from tkinter import ttk, messagebox
import json, os

DATA_FILE = "students.json"

# ── Persistence ──────────────────────────────────────────────────────────────
def load_data():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE) as f:
            return json.load(f)
    return []

def save_data(students):
    with open(DATA_FILE, "w") as f:
        json.dump(students, f, indent=2)

# ── Theme ─────────────────────────────────────────────────────────────────────
BG        = "#f5f0ff"
PRIMARY   = "#9b8ec4"
ACCENT    = "#c9b8f0"
LIGHT     = "#e8e0ff"
WHITE     = "#ffffff"
TEXT      = "#2d2d2d"
DANGER    = "#e57373"
SUCCESS   = "#81c784"
FONT      = ("Segoe UI", 11)
FONT_B    = ("Segoe UI", 11, "bold")
FONT_H    = ("Segoe UI", 16, "bold")
FONT_T    = ("Segoe UI", 20, "bold")

def styled_btn(parent, text, cmd, color=PRIMARY, fg=WHITE, width=18):
    b = tk.Button(parent, text=text, command=cmd,
                  bg=color, fg=fg, font=FONT_B,
                  relief="flat", bd=0, padx=10, pady=8,
                  cursor="hand2", width=width,
                  activebackground=ACCENT, activeforeground=WHITE)
    b.bind("<Enter>", lambda e: b.config(bg=ACCENT))
    b.bind("<Leave>", lambda e: b.config(bg=color))
    return b

def entry_field(parent, label, row, var=None):
    tk.Label(parent, text=label, bg=BG, fg=TEXT, font=FONT_B).grid(
        row=row, column=0, sticky="w", pady=4, padx=5)
    e = tk.Entry(parent, textvariable=var, font=FONT, bg=LIGHT,
                 relief="flat", bd=0, highlightthickness=1,
                 highlightbackground=PRIMARY, highlightcolor=PRIMARY, width=30)
    e.grid(row=row, column=1, pady=4, padx=5, ipady=5)
    return e

# ── Main App ──────────────────────────────────────────────────────────────────
class StudentRecordApp(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Student Record System")
        self.geometry("750x600")
        self.configure(bg=BG)
        self.resizable(True, True)
        self.students = load_data()
        self._build_main_menu()

    # ── helpers ───────────────────────────────────────────────────────────────
    def _clear(self):
        for w in self.winfo_children():
            w.destroy()

    def _header(self, parent, title):
        tk.Label(parent, text="🎓 STUDENT RECORD SYSTEM",
                 bg=PRIMARY, fg=WHITE, font=FONT_T,
                 pady=14).pack(fill="x")
        tk.Label(parent, text=title, bg=BG, fg=PRIMARY,
                 font=FONT_H, pady=8).pack()

    def _separator(self, parent):
        tk.Frame(parent, bg=ACCENT, height=2).pack(fill="x", padx=20, pady=4)

    # ── Main Menu ─────────────────────────────────────────────────────────────
    def _build_main_menu(self):
        self._clear()
        tk.Label(self, text="🎓 STUDENT RECORD SYSTEM",
                 bg=PRIMARY, fg=WHITE, font=FONT_T,
                 pady=16).pack(fill="x")

        tk.Label(self, text="Main Menu", bg=BG, fg=PRIMARY,
                 font=FONT_H, pady=10).pack()
        self._separator(self)

        frame = tk.Frame(self, bg=BG)
        frame.pack(expand=True)

        buttons = [
            ("➕  Add Student",            self._build_add_student,    PRIMARY),
            ("📋  View Students Record",    self._build_view_records,   PRIMARY),
            ("✏️   Update / Edit Record",   self._build_update_select,  PRIMARY),
            ("📊  Results",                 self._build_results,        PRIMARY),
            ("🚪  Exit",                    self._confirm_exit,         DANGER),
        ]
        for txt, cmd, col in buttons:
            styled_btn(frame, txt, cmd, color=col, width=30).pack(pady=8)

    def _confirm_exit(self):
        if messagebox.askyesno("Exit", "Are you sure you want to exit?"):
            self.destroy()

    # ── Add Student ───────────────────────────────────────────────────────────
    def _build_add_student(self):
        self._clear()
        self._header(self, "Add Student")

        canvas = tk.Canvas(self, bg=BG, highlightthickness=0)
        scroll = ttk.Scrollbar(self, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scroll.set)
        scroll.pack(side="right", fill="y")
        canvas.pack(fill="both", expand=True)

        form = tk.Frame(canvas, bg=BG, padx=30)
        canvas.create_window((0, 0), window=form, anchor="nw")
        form.bind("<Configure>", lambda e: canvas.configure(
            scrollregion=canvas.bbox("all")))

        name_v  = tk.StringVar()
        sr_v    = tk.StringVar()
        prog_v  = tk.StringVar()

        entry_field(form, "Student Name:", 0, name_v)
        entry_field(form, "SR Code:",      1, sr_v)

        # Program dropdown
        tk.Label(form, text="Program:", bg=BG, fg=TEXT, font=FONT_B).grid(
            row=2, column=0, sticky="w", pady=4, padx=5)
        programs = ["BSIT", "BSCS", "BSCE", "BSEd", "BSBA", "BSN", "BSME", "Other"]
        prog_cb = ttk.Combobox(form, textvariable=prog_v, values=programs,
                               font=FONT, width=28, state="readonly")
        prog_cb.grid(row=2, column=1, pady=4, padx=5, ipady=4)
        prog_cb.set("Select Program")

        # Dynamic subjects
        tk.Label(form, text="Subjects & Grades:", bg=BG, fg=PRIMARY,
                 font=FONT_B).grid(row=3, column=0, columnspan=2,
                                    sticky="w", pady=(12, 4), padx=5)

        subjects_frame = tk.Frame(form, bg=BG)
        subjects_frame.grid(row=4, column=0, columnspan=2, sticky="w")
        subject_rows = []

        def add_subject_row():
            r = len(subject_rows)
            subj_v = tk.StringVar()
            grade_v = tk.StringVar()
            rf = tk.Frame(subjects_frame, bg=BG)
            rf.pack(anchor="w", pady=2)
            tk.Label(rf, text=f"  Subject {r+1}:", bg=BG, font=FONT, width=10).pack(side="left")
            tk.Entry(rf, textvariable=subj_v, font=FONT, bg=LIGHT,
                     relief="flat", highlightthickness=1,
                     highlightbackground=PRIMARY, width=16).pack(side="left", padx=4, ipady=4)
            tk.Label(rf, text="Grade:", bg=BG, font=FONT).pack(side="left")
            tk.Entry(rf, textvariable=grade_v, font=FONT, bg=LIGHT,
                     relief="flat", highlightthickness=1,
                     highlightbackground=PRIMARY, width=8).pack(side="left", padx=4, ipady=4)

            def remove():
                rf.destroy()
                subject_rows.pop(r)
            if r > 0:
                tk.Button(rf, text="✕", command=remove, bg=DANGER, fg=WHITE,
                          font=FONT_B, relief="flat", padx=4).pack(side="left", padx=2)
            subject_rows.append((subj_v, grade_v))

        add_subject_row()
        styled_btn(form, "➕ Add Subject", add_subject_row,
                   color=ACCENT, fg=TEXT, width=20).grid(
            row=5, column=0, columnspan=2, pady=6, sticky="w", padx=5)

        def save_student():
            name = name_v.get().strip()
            sr   = sr_v.get().strip()
            prog = prog_v.get().strip()
            if not name or not sr or prog == "Select Program":
                messagebox.showerror("Error", "Please fill in all required fields.")
                return
            if any(s["sr_code"] == sr for s in self.students):
                messagebox.showerror("Error", "SR Code already exists!")
                return
            subjects = []
            for sv, gv in subject_rows:
                sn, gn = sv.get().strip(), gv.get().strip()
                if sn:
                    subjects.append({"subject": sn, "grade": gn})
            self.students.append({
                "name": name, "sr_code": sr,
                "program": prog, "subjects": subjects
            })
            save_data(self.students)
            messagebox.showinfo("Success", f"Student '{name}' added successfully!")
            self._build_main_menu()

        btn_f = tk.Frame(form, bg=BG)
        btn_f.grid(row=6, column=0, columnspan=2, pady=16)
        styled_btn(btn_f, "💾 Save",  save_student).pack(side="left", padx=8)
        styled_btn(btn_f, "⬅ Back",  self._build_main_menu,
                   color="#aaaaaa").pack(side="left", padx=8)

    # ── View Records ──────────────────────────────────────────────────────────
    def _build_view_records(self):
        self._clear()
        self._header(self, "View Students Record")

        search_v = tk.StringVar()
        sf = tk.Frame(self, bg=BG)
        sf.pack(fill="x", padx=20, pady=6)
        tk.Entry(sf, textvariable=search_v, font=FONT, bg=LIGHT,
                 relief="flat", highlightthickness=1,
                 highlightbackground=PRIMARY, width=35).pack(side="left", ipady=5)
        styled_btn(sf, "🔍 Search",
                   lambda: refresh(search_v.get().strip().lower()),
                   width=10).pack(side="left", padx=6)

        container = tk.Frame(self, bg=BG)
        container.pack(fill="both", expand=True, padx=20)

        canvas = tk.Canvas(container, bg=BG, highlightthickness=0)
        scroll = ttk.Scrollbar(container, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scroll.set)
        scroll.pack(side="right", fill="y")
        canvas.pack(fill="both", expand=True)

        inner = tk.Frame(canvas, bg=BG)
        canvas.create_window((0, 0), window=inner, anchor="nw")
        inner.bind("<Configure>", lambda e: canvas.configure(
            scrollregion=canvas.bbox("all")))

        def refresh(q=""):
            for w in inner.winfo_children():
                w.destroy()
            filtered = [s for s in self.students
                        if q in s["name"].lower() or q in s["sr_code"].lower()] \
                       if q else self.students
            if not filtered:
                tk.Label(inner, text="No records found.", bg=BG,
                         fg=TEXT, font=FONT).pack(pady=20)
                return
            for s in filtered:
                card = tk.Frame(inner, bg=WHITE, relief="flat",
                                bd=0, padx=14, pady=10)
                card.pack(fill="x", pady=5)
                tk.Label(card, text=f"👤 {s['name']}", bg=WHITE,
                         fg=PRIMARY, font=FONT_B).grid(row=0, column=0, sticky="w")
                tk.Label(card, text=f"🆔 {s['sr_code']}  |  📚 {s['program']}",
                         bg=WHITE, fg=TEXT, font=FONT).grid(row=1, column=0, sticky="w")
                for i, sub in enumerate(s.get("subjects", [])):
                    g = sub['grade']
                    try:
                        remark = "PASS" if float(g) <= 3.0 else "FAIL"
                        clr = SUCCESS if remark == "PASS" else DANGER
                    except:
                        remark, clr = "—", TEXT
                    tk.Label(card,
                             text=f"   • {sub['subject']} — {g}",
                             bg=WHITE, fg=TEXT, font=FONT).grid(
                        row=2+i, column=0, sticky="w")
                    tk.Label(card, text=remark, bg=clr, fg=WHITE,
                             font=FONT_B, padx=6).grid(
                        row=2+i, column=1, padx=8)

        refresh()
        styled_btn(self, "⬅ Back", self._build_main_menu,
                   color="#aaaaaa").pack(pady=8)

    # ── Update/Edit — select student ──────────────────────────────────────────
    def _build_update_select(self):
        self._clear()
        self._header(self, "Update / Edit Student Record")

        search_v = tk.StringVar()
        sf = tk.Frame(self, bg=BG)
        sf.pack(fill="x", padx=20, pady=6)
        tk.Entry(sf, textvariable=search_v, font=FONT, bg=LIGHT,
                 relief="flat", highlightthickness=1,
                 highlightbackground=PRIMARY, width=35).pack(side="left", ipady=5)
        styled_btn(sf, "🔍 Search",
                   lambda: refresh(search_v.get().strip().lower()),
                   width=10).pack(side="left", padx=6)

        container = tk.Frame(self, bg=BG)
        container.pack(fill="both", expand=True, padx=20)
        canvas = tk.Canvas(container, bg=BG, highlightthickness=0)
        scroll = ttk.Scrollbar(container, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scroll.set)
        scroll.pack(side="right", fill="y")
        canvas.pack(fill="both", expand=True)
        inner = tk.Frame(canvas, bg=BG)
        canvas.create_window((0, 0), window=inner, anchor="nw")
        inner.bind("<Configure>", lambda e: canvas.configure(
            scrollregion=canvas.bbox("all")))

        def refresh(q=""):
            for w in inner.winfo_children():
                w.destroy()
            filtered = [s for s in self.students
                        if q in s["name"].lower() or q in s["sr_code"].lower()] \
                       if q else self.students
            for s in filtered:
                card = tk.Frame(inner, bg=WHITE, padx=14, pady=10)
                card.pack(fill="x", pady=5)
                tk.Label(card, text=f"👤 {s['name']}  |  🆔 {s['sr_code']}  |  {s['program']}",
                         bg=WHITE, fg=TEXT, font=FONT_B).grid(row=0, column=0, sticky="w")
                bf = tk.Frame(card, bg=WHITE)
                bf.grid(row=0, column=1, padx=10)
                styled_btn(bf, "✏ Edit",
                           lambda st=s: self._build_edit_student(st),
                           color=PRIMARY, width=8).pack(side="left", padx=3)
                styled_btn(bf, "🗑 Delete",
                           lambda st=s: delete(st),
                           color=DANGER, width=8).pack(side="left", padx=3)

        def delete(s):
            if messagebox.askyesno("Delete",
                    f"Delete record for '{s['name']}'?"):
                self.students.remove(s)
                save_data(self.students)
                refresh()

        refresh()
        styled_btn(self, "⬅ Back", self._build_main_menu,
                   color="#aaaaaa").pack(pady=8)

    # ── Edit Student Form ─────────────────────────────────────────────────────
    def _build_edit_student(self, student):
        self._clear()
        self._header(self, "Edit Student Record")

        canvas = tk.Canvas(self, bg=BG, highlightthickness=0)
        scroll = ttk.Scrollbar(self, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scroll.set)
        scroll.pack(side="right", fill="y")
        canvas.pack(fill="both", expand=True)
        form = tk.Frame(canvas, bg=BG, padx=30)
        canvas.create_window((0, 0), window=form, anchor="nw")
        form.bind("<Configure>", lambda e: canvas.configure(
            scrollregion=canvas.bbox("all")))

        name_v = tk.StringVar(value=student["name"])
        sr_v   = tk.StringVar(value=student["sr_code"])
        prog_v = tk.StringVar(value=student["program"])

        entry_field(form, "Student Name:", 0, name_v)
        entry_field(form, "SR Code:",      1, sr_v)

        tk.Label(form, text="Program:", bg=BG, fg=TEXT, font=FONT_B).grid(
            row=2, column=0, sticky="w", pady=4, padx=5)
        programs = ["BSIT","BSCS","BSCE","BSEd","BSBA","BSN","BSME","Other"]
        prog_cb = ttk.Combobox(form, textvariable=prog_v, values=programs,
                               font=FONT, width=28, state="readonly")
        prog_cb.grid(row=2, column=1, pady=4, padx=5, ipady=4)

        tk.Label(form, text="Subjects & Grades:", bg=BG, fg=PRIMARY,
                 font=FONT_B).grid(row=3, column=0, columnspan=2,
                                    sticky="w", pady=(12,4), padx=5)
        subjects_frame = tk.Frame(form, bg=BG)
        subjects_frame.grid(row=4, column=0, columnspan=2, sticky="w")
        subject_rows = []

        def add_subject_row(subj="", grade=""):
            r = len(subject_rows)
            sv = tk.StringVar(value=subj)
            gv = tk.StringVar(value=grade)
            rf = tk.Frame(subjects_frame, bg=BG)
            rf.pack(anchor="w", pady=2)
            tk.Label(rf, text=f"  Subject {r+1}:", bg=BG, font=FONT, width=10).pack(side="left")
            tk.Entry(rf, textvariable=sv, font=FONT, bg=LIGHT,
                     relief="flat", highlightthickness=1,
                     highlightbackground=PRIMARY, width=16).pack(side="left", padx=4, ipady=4)
            tk.Label(rf, text="Grade:", bg=BG, font=FONT).pack(side="left")
            tk.Entry(rf, textvariable=gv, font=FONT, bg=LIGHT,
                     relief="flat", highlightthickness=1,
                     highlightbackground=PRIMARY, width=8).pack(side="left", padx=4, ipady=4)
            if r > 0:
                def remove(fr=rf, idx=r):
                    fr.destroy()
                    subject_rows.pop(idx)
                tk.Button(rf, text="✕", command=remove, bg=DANGER, fg=WHITE,
                          font=FONT_B, relief="flat", padx=4).pack(side="left", padx=2)
            subject_rows.append((sv, gv))

        for sub in student.get("subjects", []):
            add_subject_row(sub["subject"], sub["grade"])
        if not subject_rows:
            add_subject_row()

        styled_btn(form, "➕ Add Subject",
                   lambda: add_subject_row(),
                   color=ACCENT, fg=TEXT, width=20).grid(
            row=5, column=0, columnspan=2, pady=6, sticky="w", padx=5)

        def save_edit():
            idx = self.students.index(student)
            self.students[idx] = {
                "name": name_v.get().strip(),
                "sr_code": sr_v.get().strip(),
                "program": prog_v.get().strip(),
                "subjects": [{"subject": sv.get().strip(), "grade": gv.get().strip()}
                             for sv, gv in subject_rows if sv.get().strip()]
            }
            save_data(self.students)
            messagebox.showinfo("Success", "Record updated successfully!")
            self._build_update_select()

        bf = tk.Frame(form, bg=BG)
        bf.grid(row=6, column=0, columnspan=2, pady=16)
        styled_btn(bf, "💾 Save",   save_edit).pack(side="left", padx=8)
        styled_btn(bf, "❌ Cancel", self._build_update_select,
                   color="#aaaaaa").pack(side="left", padx=8)

    # ── Results ───────────────────────────────────────────────────────────────
    def _build_results(self):
        self._clear()
        self._header(self, "Results")

        search_v = tk.StringVar()
        sf = tk.Frame(self, bg=BG)
        sf.pack(fill="x", padx=20, pady=6)
        tk.Entry(sf, textvariable=search_v, font=FONT, bg=LIGHT,
                 relief="flat", highlightthickness=1,
                 highlightbackground=PRIMARY, width=35).pack(side="left", ipady=5)
        styled_btn(sf, "🔍 Search",
                   lambda: refresh(search_v.get().strip().lower()),
                   width=10).pack(side="left", padx=6)

        container = tk.Frame(self, bg=BG)
        container.pack(fill="both", expand=True, padx=20)
        canvas = tk.Canvas(container, bg=BG, highlightthickness=0)
        scroll = ttk.Scrollbar(container, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scroll.set)
        scroll.pack(side="right", fill="y")
        canvas.pack(fill="both", expand=True)
        inner = tk.Frame(canvas, bg=BG)
        canvas.create_window((0, 0), window=inner, anchor="nw")
        inner.bind("<Configure>", lambda e: canvas.configure(
            scrollregion=canvas.bbox("all")))

        def refresh(q=""):
            for w in inner.winfo_children():
                w.destroy()
            filtered = [s for s in self.students
                        if q in s["name"].lower() or q in s["sr_code"].lower()] \
                       if q else self.students
            if not filtered:
                tk.Label(inner, text="No records found.", bg=BG,
                         fg=TEXT, font=FONT).pack(pady=20)
                return
            for s in filtered:
                card = tk.Frame(inner, bg=WHITE, padx=14, pady=10)
                card.pack(fill="x", pady=5)
                hdr = tk.Frame(card, bg=WHITE)
                hdr.pack(fill="x")
                tk.Label(hdr, text=f"👤 {s['name']}", bg=WHITE,
                         fg=PRIMARY, font=FONT_B).pack(side="left")
                overall_pass = True
                for sub in s.get("subjects", []):
                    try:
                        if float(sub["grade"]) > 3.0:
                            overall_pass = False
                    except:
                        pass
                badge = "✅ PASSED" if overall_pass else "❌ FAILED"
                bc    = SUCCESS if overall_pass else DANGER
                tk.Label(hdr, text=badge, bg=bc, fg=WHITE,
                         font=FONT_B, padx=8).pack(side="right")

                tk.Label(card, text=f"🆔 {s['sr_code']}  |  📚 {s['program']}",
                         bg=WHITE, fg=TEXT, font=FONT).pack(anchor="w")

                if s.get("subjects"):
                    tbl = tk.Frame(card, bg=LIGHT)
                    tbl.pack(fill="x", pady=4)
                    for col, hd in enumerate(["Subject", "Grade", "Remarks"]):
                        tk.Label(tbl, text=hd, bg=PRIMARY, fg=WHITE,
                                 font=FONT_B, width=14, pady=4).grid(
                            row=0, column=col, padx=1, pady=1)
                    for r, sub in enumerate(s["subjects"], 1):
                        g = sub["grade"]
                        try:
                            remark = "PASS" if float(g) <= 3.0 else "FAIL"
                            rc = SUCCESS if remark == "PASS" else DANGER
                        except:
                            remark, rc = "—", "#aaa"
                        tk.Label(tbl, text=sub["subject"], bg=WHITE,
                                 font=FONT, width=14, pady=3).grid(
                            row=r, column=0, padx=1, pady=1)
                        tk.Label(tbl, text=g, bg=WHITE,
                                 font=FONT, width=14).grid(
                            row=r, column=1, padx=1, pady=1)
                        tk.Label(tbl, text=remark, bg=rc,
                                 fg=WHITE, font=FONT_B, width=14).grid(
                            row=r, column=2, padx=1, pady=1)

        refresh()
        styled_btn(self, "⬅ Back", self._build_main_menu,
                   color="#aaaaaa").pack(pady=8)

# ── Run ───────────────────────────────────────────────────────────────────────
if __name__ == "__main__":
    app = StudentRecordApp()
    app.mainloop()

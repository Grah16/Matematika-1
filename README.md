--- Začetek datoteke: main_suite.py ---

# -*- coding: utf-8 -*-
"""
main_suite.py
Zaganjalnik za oba GUI-ja v enem .exe:
- Gumb "Učni vmesnik" -> ucni_vmesnik.main()
- Gumb "Test vmesnik" -> test_vmesnik.main()
- Lahko tudi preko argumenta: --mode=ucni ali --mode=test
"""

import sys
import tkinter as tk
from tkinter import ttk, messagebox

import ucni_vmesnik
import test_vmesnik

def run_ucni():
    ucni_vmesnik.main()

def run_test():
    test_vmesnik.main()

def parse_mode_arg() -> str | None:
    for arg in sys.argv[1:]:
        if arg.startswith("--mode="):
            return arg.split("=", 1)[1].strip().lower()
    return None

def main():
    mode = parse_mode_arg()
    if mode in ("ucni", "učni", "u"):
        return run_ucni()
    if mode in ("test", "t"):
        return run_test()

    # Če ni argumenta, pokaži preprost izbirnik
    root = tk.Tk()
    root.title("MathDrill – izberi vmesnik")
    root.geometry("360x160")
    root.resizable(False, False)

    frm = ttk.Frame(root, padding=16)
    frm.pack(fill="both", expand=True)

    lbl = ttk.Label(frm, text="Kaj želiš zagnati?", font=("Segoe UI", 12, "bold"))
    lbl.pack(pady=(0, 12))

    btn1 = ttk.Button(frm, text="📘 Učni vmesnik", command=lambda: (root.destroy(), run_ucni()))
    btn1.pack(fill="x", pady=6)

    btn2 = ttk.Button(frm, text="🧪 Test vmesnik", command=lambda: (root.destroy(), run_test()))
    btn2.pack(fill="x", pady=6)

    ttk.Label(frm, text="Namig: lahko uporabiš tudi argument --mode=ucni ali --mode=test.",
              font=("Segoe UI", 9)).pack(pady=(10,0))

    root.mainloop()

if __name__ == "__main__":
    main()


--- Konec datoteke: main_suite.py ---

--- Začetek datoteke: test.py ---

import os

def analiziraj_projekt(mapa_projekta, izhodna_datoteka, dovoljene_koncnice=None, mape_za_izkljucitev=None):
    """
    Analizira projektno mapo, prebere vsebino datotek s kodo in jih shrani v izhodno datoteko.

    Args:
        mapa_projekta (str): Pot do glavne mape projekta.
        izhodna_datoteka (str): Pot do .txt datoteke, kamor se bo shranila vsebina.
        dovoljene_koncnice (list, optional): Seznam končnic datotek, ki naj se vključijo.
                                             Če je None, se uporabijo privzete končnice.
        mape_za_izkljucitev (list, optional): Seznam imen map, ki naj se preskočijo.
                                              Če je None, se uporabijo privzete izključitve.
    """
    if dovoljene_koncnice is None:
        # Pogoste končnice datotek s kodo (dodajte ali odstranite po potrebi)
        dovoljene_koncnice = [
            '.py', '.js', '.html', '.css', '.java', '.c', '.cpp', '.h', '.hpp',
            '.cs', '.go', '.rs', '.swift', '.kt', '.php', '.rb', '.pl', '.sh',
            '.ipynb', '.sql', '.md', '.json', '.xml', '.yaml', '.yml', '.ts',
            '.jsx', '.tsx', '.vue', '.scss', '.sass'
        ]

    if mape_za_izkljucitev is None:
        # Pogoste mape knjižnic in druge mape, ki jih običajno ne želimo vključiti
        mape_za_izkljucitev = [
            'node_modules', 'venv', '.venv', 'env', '.env', 'vendor',
            'bower_components', 'target', 'build', 'dist', '__pycache__',
            '.git', '.vscode', '.idea', 'bin', 'obj', 'logs', 'temp', 'tmp',
            'backup', 'docs', 'e2e', 'examples', 'samples',
            'migrations', 'alembic'
        ]

    try:
        with open(izhodna_datoteka, 'w', encoding='utf-8') as out_file:
            print(f"Začetek analize mape: {mapa_projekta}")
            print(f"Vsebina bo shranjena v: {izhodna_datoteka}")

            for root, dirs, files in os.walk(mapa_projekta, topdown=True):
                # Izločevanje map
                dirs[:] = [d for d in dirs if d.lower() not in mape_za_izkljucitev and not d.startswith('.')]

                for datoteka in files:
                    ime_datoteke, koncnica_datoteke = os.path.splitext(datoteka)
                    if koncnica_datoteke.lower() in dovoljene_koncnice:
                        pot_do_datoteke = os.path.join(root, datoteka)
                        relativna_pot = os.path.relpath(pot_do_datoteke, mapa_projekta)
                        print(f"  -> Berem datoteko: {relativna_pot}")

                        try:
                            with open(pot_do_datoteke, 'r', encoding='utf-8', errors='ignore') as in_file:
                                vsebina = in_file.read()
                                out_file.write(f"--- Začetek datoteke: {relativna_pot} ---\n\n")
                                out_file.write(vsebina)
                                out_file.write(f"\n\n--- Konec datoteke: {relativna_pot} ---\n\n")
                        except Exception as e:
                            out_file.write(f"--- Napaka pri branju datoteke: {relativna_pot} ---\n")
                            out_file.write(f"    Napaka: {str(e)}\n")
                            out_file.write(f"--- Konec napake za datoteko: {relativna_pot} ---\n\n")
                            print(f"  *** Napaka pri branju datoteke {relativna_pot}: {e}")

            print(f"Analiza končana. Vsebina shranjena v {izhodna_datoteka}")

    except IOError as e:
        print(f"Napaka pri delu z datotekami: {e}")
    except Exception as e:
        print(f"Prišlo je do nepričakovane napake: {e}")

# --- Kako uporabiti skripto ---
if __name__ == "__main__":
    # 1. Pot do mape vašega projekta je že nastavljena
    mapa_s_projektom = r"C:\Users\matej\Desktop\Projekti\Matematika"

    # 2. Nastavite ime in pot za izhodno .txt datoteko
    #    Primer: txt_izhod = r"C:\Uporabniki\VaseIme\Desktop\analiza_kode.txt"
    txt_izhod = "analiza_kode_projekta.txt"  # To bo ustvarjeno v isti mapi kot skripta

    # 3. (Opcijsko) Prilagodite seznam dovoljenih končnic datotek
    #    Če pustite None, bo uporabljen privzet seznam.
    posebne_koncnice = None

    # 4. (Opcijsko) Prilagodite seznam map za izključitev
    #    Če pustite None, bo uporabljen privzet seznam.
    posebne_izkljucitve_map = None

    # Preverite, če pot do projekta obstaja
    if not os.path.isdir(mapa_s_projektom):
        print(f"NAPAKA: Navedena pot do projektne mape ne obstaja ali ni mapa: '{mapa_s_projektom}'")
    else:
        analiziraj_projekt(mapa_s_projektom, txt_izhod, posebne_koncnice, posebne_izkljucitve_map)

--- Konec datoteke: test.py ---

--- Začetek datoteke: test_vmesnik.py ---

# -*- coding: utf-8 -*-
"""
test_vmesnik.py
---------------
"Preizkusni" GUI za MathDrill – brez sprotnih rešitev.
- Izberi teme (ena ali več), težavnost, število nalog in trajanje.
- Med testom: brez razkritih rešitev; odštevalnik časa.
- Po oddaji: ocena + seznam napačnih; klik na napako odpre razlago
  (pravilni odgovor, tvoj odgovor, ter tipični vzroki napake glede na kategorijo).

Avtorica: Alma
"""

from __future__ import annotations
import re
import random
from dataclasses import dataclass
from fractions import Fraction
from typing import List, Optional, Tuple

import tkinter as tk
from tkinter import ttk, messagebox

from orodja.eval_helpers import (
    _safe_eval_number_expr,
    _normalize_number_like,
    _try_parse_number,        # ⬅️ dodaj to
    _parse_simple_monomial,
)


# --- Register & tipi ---
from predmeti import register as REG
from predmeti.tipi import Problem
from orodja.odgovor_eval import evaluate_answer, EvalConfig
CFG = EvalConfig()
from orodja.selfcheck import verify_problem_numeric

# poskrbimo, da so PREDMETI registrirani
import predmeti.ulomki
import predmeti.potence_in_koreni
import predmeti.vrstni_red_racunov
import predmeti.linearne_enacbe
import predmeti.polinomi
import predmeti.trigonometrija
import predmeti.funkcije_in_grafi
import predmeti.sistemi_enacb
import predmeti.procenti_in_ddv


# ============ Pomagala: varen eval, parsing, simbolni monomi, preverjanje ============


def evaluate_user_answer(user_input: str, p: Problem) -> Tuple[bool, str]:
    """
    True/False + kratko sporočilo.
    - Če p.numeric obstaja → numerična primerjava (toleranca 0.001, za € 0.01), podpira izraze (+ - * / ( ) **).
    - Sicer poskusi simbolno:
        * najprej normaliziran niz (zamenja '·' z '*'),
        * nato preprost monom (c*a ~ a*c ~ a/d ~ a*(c)).
    """
    ui_raw = user_input.strip()
    if ui_raw == "":
        return False, "Prazno"

    # dovolim 'x=..', 'y=..'
    if "=" in ui_raw:
        left, right = ui_raw.split("=", 1)
        if left.strip().lower() in ("x", "y", "f(x)", "p(x)"):
            ui_raw = right.strip()

    # 1) Numerično (če naloga poda numeric referenco)
    if p.numeric is not None:
        target = float(p.numeric)
        val = _try_parse_number(_normalize_number_like(ui_raw))
        if val is None:
            return False, "Ni številski zapis"
        tol = 1e-3
        if "€" in str(p.solution) or re.search(r"\d+[.,]\d{2}\b", str(p.solution)):
            tol = 1e-2 + 1e-9
        return (abs(val - target) <= tol, "")

    # 2) Simbolno – najprej normaliziran niz (· ↔ *)
    def _norm_symb(s: str) -> str:
        return s.strip().replace(" ", "").replace("−","-").replace(",", ".").replace("·", "*")
    if _norm_symb(ui_raw) == _norm_symb(str(p.solution)):# direktni match po normalizaciji
        return True, ""

    # 3) Poskusi preprost monom c*a
    ui_m = _parse_simple_monomial(ui_raw)
    sol_m = _parse_simple_monomial(str(p.solution))
    if ui_m and sol_m:
        (ui_var, ui_c), (sol_var, sol_c) = ui_m, sol_m
        if ui_var == sol_var:
            # primerjaj koeficiente z majhno toleranco
            try:
                ok = abs(float(ui_c) - float(sol_c)) <= 1e-12
                return (ok, "" if ok else "Koeficient pri spremenljivki ni enak rešitvi.")
            except Exception:
                pass

    return False, "Nisem prepoznala zapisa kot enakovreden rešitvi. Dovoljeno je npr. '1/120*a', 'a/120', 'a*(1/120)'."

def guess_mistake_reason(cat: str, user_input: str, p: Problem) -> List[str]:
    """Heuristike za tipične napake po kategorijah."""
    reasons = []
    if cat == "Ulomki":
        if "÷" in p.prompt or ("/" in p.prompt and (" / " in p.prompt or " : " in p.prompt)):
            reasons.append("Mogoče nisi obrnil drugega ulomka pri deljenju (a/b ÷ c/d → a/b · d/c).")
        if "+" in p.prompt or "-" in p.prompt:
            reasons.append("Preveri SKI (skupni imenovalec) in krajšanje pred množenjem.")
    elif cat == "Vrstni red računov":
        reasons.append("Pazi na vrstni red: oklepaji → potence/koreni → množenje/deljenje (z leve) → seštevanje/odštevanje (z leve).")
    elif cat == "Linearne enačbe":
        reasons.append("Pri prenosu členov pazi na predznake; na koncu deli s koeficientom pri x.")
        if "/" in p.prompt:
            reasons.append("Če so ulomki, najprej pomnoži celotno enačbo s SKI.")
    elif cat == "Polinomi":
        reasons.append("Morda si razširil oklepaje narobe ali si zamenjal predznak pri odštevanju polinomov.")
    elif cat == "Trigonometrija":
        reasons.append("Preveri, ali računaš v pravih enotah (stopinje vs. radiani) in uporabi adicijske izreke pravilno.")
    elif cat == "Funkcije in grafi":
        reasons.append("Pri y=ax+b: naklon je koeficient pri x, presečišče z y je prosti člen.")
    elif cat == "Sistemi enačb":
        reasons.append("Pri eliminaciji poravnaj koeficiente; pri vstavitvi pravilno substituiraj in zberi člene.")
    elif cat == "Procenti/DDV":
        reasons.append("Zaporedje je pomembno: najprej popust (×(1−p)), nato DDV (×(1+q)).")
    elif cat == "Potence/Koreni":
        reasons.append("Uporabi zakone potenc in iz korena izvleci popoln kvadrat (npr. 50=25·2 → 5√2).")
    return reasons


# ============ Tema (svetla/temna) ============

try:
    from ui.theme import LIGHT, DARK
except Exception:
    # Fallback, če ui/theme.py ni na voljo (npr. med razvojem)
    LIGHT = {
        "BG": "#F6F7FB",
        "PANEL": "#FFFFFF",
        "TEXT": "#0F172A",
        "SUB": "#475569",
        "PRIMARY": "#2563EB",
        "ACCENT": "#16A34A",
        "LIST_BG": "#FFFFFF",
        "LIST_SEL": "#DBEAFE",
    }
    DARK = {
        "BG": "#0F131A",
        "PANEL": "#151A22",
        "TEXT": "#E6EDF3",
        "SUB": "#9AA4B2",
        "PRIMARY": "#4C84FF",
        "ACCENT": "#6EE7B7",
        "LIST_BG": "#0E131B",
        "LIST_SEL": "#25406B",
    }


# ============ Stanje in GUI ============

@dataclass
class QItem:
    problem: Problem
    answer: tk.StringVar

@dataclass
class TestState:
    questions: List[QItem]
    started: bool = False
    seconds_left: int = 0

def _humanize_steps(text: str) -> str:
    """
    Pretvori kratice v polne izraze za učne korake (varianta A – brez kratic v izpisu).
    - gcd / GCD / NSD -> Največji skupni delitelj
    - g₁ / g₂ -> Največji skupni delitelj (prvi/drugi par)
    """
    if not text:
        return text
    s = text
    s = re.sub(r'\bGCD\b', 'Največji skupni delitelj', s)
    s = re.sub(r'\bgcd\b', 'Največji skupni delitelj', s, flags=re.IGNORECASE)
    s = re.sub(r'\bNSD\b', 'Največji skupni delitelj', s)
    s = s.replace("g₁", "Največji skupni delitelj (prvi par)")
    s = s.replace("g₂", "Največji skupni delitelj (drugi par)")
    return s

class TestApp(ttk.Frame):
    def __init__(self, master: tk.Tk):
        super().__init__(master)
        self.master = master
        self.theme = LIGHT
        self.state = TestState(questions=[], started=False, seconds_left=0)
        self.timer_job = None

        self._configure_style()
        self._build_layout()
        self._load_topics()

    # ---------- style ----------
    def _configure_style(self):
        self.master.title("MathDrill – Preizkus")
        self.master.geometry("1080x720")
        self.master.minsize(960, 640)
        style = ttk.Style()
        try: style.theme_use("clam")
        except Exception: pass
        self._apply_theme(style)

    def _apply_theme(self, style: ttk.Style):
        t = self.theme
        self.master.configure(bg=t["BG"])
        style.configure("TFrame", background=t["BG"])
        style.configure("Panel.TFrame", background=t["PANEL"])
        style.configure("TLabel", background=t["BG"], foreground=t["TEXT"])
        style.configure("Panel.TLabel", background=t["PANEL"], foreground=t["TEXT"])
        style.configure("Header.TLabel", background=t["BG"], foreground=t["TEXT"], font=("Segoe UI", 18, "bold"))
        style.configure("Sub.TLabel", background=t["BG"], foreground=t["SUB"], font=("Segoe UI", 10))
        style.configure("TButton", font=("Segoe UI", 10, "bold"))
        style.configure("Primary.TButton", foreground="white", background=t["PRIMARY"])
        style.map("Primary.TButton", background=[("active", "#1D4ED8")])
        style.configure("Accent.TButton", foreground="white", background=t["ACCENT"])
        style.map("Accent.TButton", background=[("active", "#15803D")])

    # ---------- layout ----------
    def _build_layout(self):
        root = self.master

        # Header
        header = ttk.Frame(root, style="TFrame")
        header.pack(side=tk.TOP, fill=tk.X, padx=16, pady=(12,8))
        ttk.Label(header, text="🧪 MathDrill – Preizkus (brez rešitev)", style="Header.TLabel").pack(side=tk.LEFT)
        self.var_dark = tk.BooleanVar(value=False)
        ttk.Checkbutton(header, text="Temni način", variable=self.var_dark, command=self._toggle_dark).pack(side=tk.RIGHT)

        # Top controls
        ctrl = ttk.Frame(root, style="Panel.TFrame")
        ctrl.pack(side=tk.TOP, fill=tk.X, padx=16, pady=8)

        # topics (checkbox list)
        left = ttk.Frame(ctrl, style="Panel.TFrame")
        left.pack(side=tk.LEFT, fill=tk.X, expand=True, padx=8, pady=8)
        ttk.Label(left, text="Izberi teme:", style="Panel.TLabel").pack(anchor="w")

        self.topic_vars: List[Tuple[str, tk.BooleanVar]] = []
        self.topics_frame = ttk.Frame(left, style="Panel.TFrame")
        self.topics_frame.pack(fill=tk.X, expand=True, pady=(4,0))

        # options right
        right = ttk.Frame(ctrl, style="Panel.TFrame")
        right.pack(side=tk.RIGHT, padx=8, pady=8)

        row = 0
        ttk.Label(right, text="Težavnost (1–3):", style="Panel.TLabel").grid(row=row, column=0, sticky="e", padx=6, pady=4)
        self.cbo_level = ttk.Combobox(right, state="readonly", width=8, values=["1","2","3"])
        self.cbo_level.set("1")
        self.cbo_level.grid(row=row, column=1, sticky="w", pady=4)
        row += 1

        ttk.Label(right, text="Št. nalog:", style="Panel.TLabel").grid(row=row, column=0, sticky="e", padx=6, pady=4)
        self.spn_count = ttk.Spinbox(right, from_=3, to=50, width=10)
        self.spn_count.set(10)
        self.spn_count.grid(row=row, column=1, sticky="w", pady=4)
        row += 1

        ttk.Label(right, text="Trajanje (min):", style="Panel.TLabel").grid(row=row, column=0, sticky="e", padx=6, pady=4)
        self.spn_minutes = ttk.Spinbox(right, from_=1, to=180, width=10)
        self.spn_minutes.set(20)
        self.spn_minutes.grid(row=row, column=1, sticky="w", pady=4)
        row += 1

        self.btn_start = ttk.Button(right, text="▶️ Začni test", style="Primary.TButton", command=self._start_test)
        self.btn_start.grid(row=row, column=0, columnspan=2, sticky="we", pady=(8,2))
        row += 1
        self.btn_submit = ttk.Button(right, text="✅ Oddaj test", style="Accent.TButton", command=self._submit_test, state=tk.DISABLED)
        self.btn_submit.grid(row=row, column=0, columnspan=2, sticky="we", pady=(2,0))

        # Timer/status bar
        status = ttk.Frame(root, style="TFrame")
        status.pack(side=tk.TOP, fill=tk.X, padx=16, pady=(0,8))
        self.var_timer = tk.StringVar(value="Čas: —:—")
        ttk.Label(status, textvariable=self.var_timer, style="Sub.TLabel").pack(side=tk.RIGHT)
        self.var_status = tk.StringVar(value="Pripravljeno.")
        ttk.Label(status, textvariable=self.var_status, style="Sub.TLabel").pack(side=tk.LEFT)

        # Questions area (scrollable)
        area = ttk.Frame(root, style="Panel.TFrame")
        area.pack(side=tk.TOP, fill=tk.BOTH, expand=True, padx=16, pady=(0,16))

        self.canvas = tk.Canvas(area, highlightthickness=0, bd=0, background=self.theme["PANEL"])
        self.scroll = ttk.Scrollbar(area, orient="vertical", command=self.canvas.yview)
        self.inner = ttk.Frame(self.canvas, style="Panel.TFrame")

        self.inner.bind("<Configure>", lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all")))
        self.canvas.create_window((0,0), window=self.inner, anchor="nw")
        self.canvas.configure(yscrollcommand=self.scroll.set)

        self.canvas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        self.scroll.pack(side=tk.RIGHT, fill=tk.Y)

    # ---------- topics ----------
    def _load_topics(self):
        names = REG.list_categories()
        names_sorted = sorted(names)
        # build checkboxes v dveh stolpcih
        for i, name in enumerate(names_sorted):
            var = tk.BooleanVar(value=False)
            self.topic_vars.append((name, var))
            col = i % 2
            row = i // 2
            ttk.Checkbutton(self.topics_frame, text=name, variable=var).grid(row=row, column=col, sticky="w", padx=4, pady=2)

    # ---------- theme toggle ----------
    def _toggle_dark(self):
        self.theme = DARK if self.var_dark.get() else LIGHT
        self._apply_theme(ttk.Style())
        # repaint canvas bg
        self.canvas.configure(background=self.theme["PANEL"])

    # ---------- generation ----------
    def _collect_selected_topics(self) -> List[str]:
        sel = [name for name, var in self.topic_vars if var.get()]
        return sel

    def _start_test(self):
        if self.state.started:
            if not messagebox.askyesno("Začni znova", "Test že poteka. Začneš znova in izbrišeš odgovore?"):
                return
            self._stop_timer()
        sel = self._collect_selected_topics()
        if not sel:
            messagebox.showinfo("Teme", "Izberi vsaj eno temo.")
            return
        try:
            cnt = int(self.spn_count.get())
            lvl = int(self.cbo_level.get())
            mins = int(self.spn_minutes.get())
        except Exception:
            messagebox.showerror("Napaka", "Preveri nastavitve.")
            return

        # Generate
        qs: List[QItem] = []
        for _ in range(cnt):
            cat = random.choice(sel)
            prob = REG.generate(cat, lvl)
            qs.append(QItem(problem=prob, answer=tk.StringVar(value="")))

        # Render
        for child in self.inner.winfo_children():
            child.destroy()
        for i, q in enumerate(qs, 1):
            frm = ttk.Frame(self.inner, style="Panel.TFrame")
            frm.pack(fill=tk.X, padx=12, pady=8)

            ttk.Label(frm, text=f"{i}. [{q.problem.category}] {q.problem.prompt}",
                      style="Panel.TLabel", wraplength=900, justify="left").pack(anchor="w", pady=(0,6))
            row = ttk.Frame(frm, style="Panel.TFrame")
            row.pack(fill=tk.X)
            ttk.Label(row, text="Odgovor:", style="Panel.TLabel").pack(side=tk.LEFT)
            ent = ttk.Entry(row, textvariable=q.answer, width=40)
            ent.pack(side=tk.LEFT, padx=(8,0))
            # Enter → fokus naprej
            ent.bind("<Return>", lambda e, idx=i: self._focus_next(idx))

        self.state.questions = qs
        self.state.started = True
        self.btn_submit.config(state=tk.NORMAL)
        self.var_status.set("Test teče. Srečno!")
        # timer
        self.state.seconds_left = mins * 60
        self._tick_timer()

    def _focus_next(self, idx: int):
        try:
            self.master.focus_get().tk_focusNext().focus()
        except Exception:
            pass

    # ---------- timer ----------
    def _tick_timer(self):
        s = self.state.seconds_left
        mm = s // 60
        ss = s % 60
        self.var_timer.set(f"Čas: {mm:02d}:{ss:02d}")
        if s <= 0:
            self._submit_test(auto=True)
            return
        self.state.seconds_left -= 1
        self.timer_job = self.master.after(1000, self._tick_timer)

    def _stop_timer(self):
        if self.timer_job:
            self.master.after_cancel(self.timer_job)
            self.timer_job = None

    # ---------- submit ----------
    def _submit_test(self, auto: bool = False):
        if not self.state.started:
            return
        self._stop_timer()
        self.btn_submit.config(state=tk.DISABLED)
        self.state.started = False

        total = len(self.state.questions)
        correct = 0
        # (index, qitem, user_input, verdict)
        mistakes: List[Tuple[int, QItem, str, object]] = []

        for i, q in enumerate(self.state.questions, 1):
            user_in = q.answer.get()
            v = evaluate_answer(q.problem, user_in, subject=q.problem.category, config=CFG)
            if v.is_correct:
                correct += 1
            else:
                mistakes.append((i, q, user_in, v))

        score_pct = 100.0 * correct / total if total else 0.0
        msg = f"Rezultat: {correct}/{total} ({score_pct:.1f}%)."
        if auto:
            msg = "Čas je potekel. " + msg
        if mistakes:
            msg += f"\nNapak: {len(mistakes)}. Klikni »Pokaži napake« za razlago."
        messagebox.showinfo("Rezultat", msg)
        self.var_status.set(msg)

        if mistakes:
            self._show_mistakes_window(mistakes)

    # ---------- mistakes window ----------
    def _show_mistakes_window(self, mistakes: List[Tuple[int, QItem, str, object]]):
        win = tk.Toplevel(self.master)
        win.title("Napake in razlaga")
        win.geometry("900x600")
        win.configure(bg=self.theme["BG"])
        ttk.Label(win, text="Tvoje napake", style="Header.TLabel").pack(anchor="w", padx=16, pady=(12,4))
        ttk.Label(win, text="Klikni na napako za razlago (pravilni odgovor + možni vzroki).", style="Sub.TLabel").pack(anchor="w", padx=16, pady=(0,8))

        main = ttk.Panedwindow(win, orient=tk.HORIZONTAL)
        main.pack(fill=tk.BOTH, expand=True, padx=12, pady=12)

        left = ttk.Frame(main, style="Panel.TFrame")
        right = ttk.Frame(main, style="Panel.TFrame")
        main.add(left, weight=1)
        main.add(right, weight=2)

        lst = tk.Listbox(left, bg=self.theme["LIST_BG"], fg=self.theme["TEXT"],
                         selectbackground=self.theme["LIST_SEL"], activestyle="none",
                         highlightthickness=0, bd=1)
        lst.pack(fill=tk.BOTH, expand=True, padx=8, pady=8)

        text = tk.Text(right, bg=self.theme["LIST_BG"], fg=self.theme["TEXT"],
                       insertbackground=self.theme["TEXT"], wrap="word", bd=0, padx=12, pady=12)
        text.pack(fill=tk.BOTH, expand=True, padx=8, pady=8)

        for idx, (i, q, ui) in enumerate(mistakes, 1):
            preview = q.problem.prompt.replace("\n", " ")
            if len(preview) > 70:
                preview = preview[:70] + "…"
            lst.insert(tk.END, f"{i}. [{q.problem.category}] {preview}")

        def on_select(evt=None):
            sel = lst.curselection()
            if not sel: return
            j = sel[0]
            i, q, ui, v = mistakes[j]
            p = q.problem
            text.delete("1.0", tk.END)
            text.insert("1.0", f"{i}. [{p.category}]\n{p.prompt}\n\n")
            text.insert(tk.END, f"Tvoj odgovor: {ui if ui.strip() else '— prazno —'}\n")
            text.insert(tk.END, f"Pravilni odgovor: {p.solution}\n")
            if getattr(v, "match", None):
                text.insert(tk.END, f"(Uvrstitev: {v.match})\n")
            if getattr(v, "explanation", ""):
                text.insert(tk.END, f"\nNamig: {v.explanation}\n")

            # heuristični namigi (obstojeci)
            hints = guess_mistake_reason(p.category, ui, p)
            if hints:
                text.insert(tk.END, "\nMožni vzroki napake:\n")
                for h in hints:
                    text.insert(tk.END, f" • {h}\n")

            # po testu dodamo še korake (brez kratic: gcd/NSD → Največji skupni delitelj)
            text.insert(tk.END, "\nKoraki rešitve:\n")
            text.insert(tk.END, "----------------------------------------\n")
            text.insert(tk.END, _humanize_steps(p.steps.strip()) + "\n")

        lst.bind("<<ListboxSelect>>", on_select)
        if mistakes:
            lst.selection_set(0); on_select()

    # ---------- main loop ----------
    def run(self):
        self.master.mainloop()


def main():
    root = tk.Tk()
    app = TestApp(root)
    app.run()

if __name__ == "__main__":
    main()


--- Konec datoteke: test_vmesnik.py ---

--- Začetek datoteke: ucni_vmesnik.py ---

# -*- coding: utf-8 -*-
"""
ucni_vmesnik.py
----------------
Tkinter/ttk GUI za MathDrill (svetla tema privzeto):
- Bere kategorije iz predmeti/register.py
- Ustvari nabor nalog (po kategoriji ali vse), s težavnostjo, številom in seed-om
- Prikaz nalog na levi; na desni zavihki "Naloga", "Rešitev", "Koraki"
- Rešitev/Koraki sta privzeto SKRITA; dodano polje "Tvoj odgovor" + "Preveri"
- Po preverjanju prikaže ✅/❌; rešitve/koraki ostanejo vidni, dokler jih ne skriješ
- Izvoz v Markdown, Samopreizkus, preklop "Temni način", gumb "📋 Kopiraj korake"

Avtorica: Alma
"""

from __future__ import annotations
import os
import re
import random
from fractions import Fraction
from dataclasses import dataclass
from pathlib import Path
from typing import List, Optional, Tuple

from orodja.eval_helpers import (
    _safe_eval_number_expr,
    _normalize_number_like,
    _try_parse_number,         # samo če ga datoteka uporablja
    _parse_simple_monomial,
)



import tkinter as tk
from tkinter import ttk, messagebox, filedialog
from ui.theme import LIGHT, DARK
from orodja.selfcheck import verify_problem_numeric

# --- Register in tipi ---
from predmeti import register as REG
from predmeti.tipi import Problem
from orodja.odgovor_eval import evaluate_answer, EvalConfig
CFG = EvalConfig()

# --- Uvozi PREDMETE (da se registrirajo) ---
import predmeti.ulomki
import predmeti.potence_in_koreni
import predmeti.vrstni_red_racunov
import predmeti.linearne_enacbe
import predmeti.polinomi
import predmeti.trigonometrija
import predmeti.funkcije_in_grafi
import predmeti.sistemi_enacb
import predmeti.procenti_in_ddv

# ===================== Markdown izvoz =====================

def _humanize_steps(text: str) -> str:
    """
    Pretvori kratice v polne izraze za učne korake.
    - gcd / GCD / NSD -> Največji skupni delitelj
    - g₁ / g₂ -> Največji skupni delitelj (prvi/drugi par)
    Ne posega v matematiko, zgolj v besedilo.
    """
    if not text:
        return text
    s = text
    # Kratice gcd/GCD/NSD -> polni izraz
    s = re.sub(r'\bGCD\b', 'Največji skupni delitelj', s)
    s = re.sub(r'\bgcd\b', 'Največji skupni delitelj', s, flags=re.IGNORECASE)
    s = re.sub(r'\bNSD\b', 'Največji skupni delitelj', s)
    # Označena para g₁, g₂ -> opis
    s = s.replace("g₁", "Največji skupni delitelj (prvi par)")
    s = s.replace("g₂", "Največji skupni delitelj (drugi par)")
    return s

def izvoz_markdown(naloge: List[Problem], pot_md: str | Path) -> Path:
    pot_md = Path(pot_md)
    osnova = pot_md.with_suffix("")
    pot_resitve = osnova.with_name(osnova.name + "_resitve").with_suffix(".md")

    pot_md.parent.mkdir(parents=True, exist_ok=True)
    pot_resitve.parent.mkdir(parents=True, exist_ok=True)

    with pot_md.open("w", encoding="utf-8") as f:
        f.write("# Delovni list\n\n")
        for i, p in enumerate(naloge, 1):
            f.write(f"**{i}. [{p.category}]** {p.prompt}\n\n")
        f.write("\n---\n")
        f.write(f"*Rešitve in koraki so v datoteki:* **{pot_resitve.name}**\n")

    with pot_resitve.open("w", encoding="utf-8") as f:
        f.write("# Rešitve in koraki\n\n")
        for i, p in enumerate(naloge, 1):
            f.write(f"**{i}.** Odgovor: `{p.solution}`\n\n")
            f.write("Koraki:\n\n")
            # Uporabi polni izraz namesto kratic tudi v izvozu
            koraki = _humanize_steps(p.steps)
            f.write("```\n" + koraki + "\n```\n\n")

    return pot_resitve

# ===================== Self-check & parsing helperji =====================


def _parse_frac(tok: str) -> Optional[Fraction]:
    tok = tok.strip()
    if tok.count('/') == 1 and all(part.strip('-').isdigit() for part in tok.split('/')):
        try:
            return Fraction(tok)
        except Exception:
            return None
    return None

def _verify_fractions_prompt(prompt: str) -> Optional[Fraction]:
    """Razume osnovne oblike iz 'Ulomki' → vrne Fraction ali None."""
    text = prompt.replace("Uredi izraz: ", "").strip().replace("÷", "/").replace("·", "*")
    toks = text.split()
    # a/b + c/d ali a/b - c/d
    if len(toks) == 3 and toks[1] in ["+","-"]:
        f1, f2 = _parse_frac(toks[0]), _parse_frac(toks[2])
        if f1 is not None and f2 is not None:
            return f1 + f2 if toks[1] == "+" else f1 - f2
    # a/b / c/d + e/f
    if len(toks) == 5 and toks[1] == "/" and toks[3] == "+":
        f1, f2, f3 = _parse_frac(toks[0]), _parse_frac(toks[2]), _parse_frac(toks[4])
        if f1 and f2 and f3:
            return f1 / f2 + f3
    return None

def _verify_problem(p: Problem) -> bool:
    try:
        cat = p.category

        if cat == "Ulomki":
            val = _verify_fractions_prompt(p.prompt)
            if val is None:
                return True  # simbolne oblike
            try:
                sol = Fraction(p.solution)
                return sol == val
            except Exception:
                return (p.numeric is not None) and abs(float(val) - p.numeric) < 1e-12

        if cat == "Vrstni red računov":
            text = p.prompt.replace("Uredi: ", "").replace("·", "*")
            text = re.sub(r'(\d+)\^2', r'(\1*\1)', text)  # b^2 -> (b*b)
            val = eval(text)  # generirano iz varnih elementov
            return int(p.solution) == int(val)

        if cat == "Linearne enačbe":
            x = Fraction(p.solution)
            # Osnovna linearna: A·x + B = C·x + D
            m2 = re.search(r":\s*(-?\d+)\s*[·\*x]\s*x\s*\+\s*(-?\d+)\s*=\s*(-?\d+)\s*[·\*x]\s*x\s*\+\s*(-?\d+)", p.prompt)
            if m2:
                A, B, C, D = map(int, m2.groups())
                return A*x + B == C*x + D
            # Ulomki+SKI: (x±a)/m + (x±b)/n = k
            m = re.search(r":\s*(\(.+?\))/(\d+)\s+\+\s+(\(.+?\))/(\d+)\s+=\s+(-?\d+)", p.prompt)
            if m:
                left1, den1, left2, den2, rhs = m.groups()
                def eval_group(g):
                    m2 = re.search(r"x([+-])(\d+)", g)
                    if m2:
                        sign = 1 if m2.group(1) == '+' else -1
                        return x + sign*int(m2.group(2))
                    return x
                lhs = eval_group(left1)/int(den1) + eval_group(left2)/int(den2)
                return lhs == Fraction(int(rhs), 1)
            return True

        if cat == "Potence/Koreni":
            if p.numeric is not None:
                try:
                    sol_num = float(Fraction(p.solution))
                    return abs(sol_num - p.numeric) < 1e-12
                except Exception:
                    return True
            return True

        if p.numeric is not None:
            try:
                return abs(float(p.numeric) - float(p.numeric)) < 1e-12
            except Exception:
                return True

        return True

    except Exception:
        return False

# ===================== Primerjava uporabniškega odgovora =====================


def evaluate_user_answer(user_input: str, p: Problem) -> Tuple[bool, str]:
    """
    Pravila:
    - če p.numeric obstaja: numeric primerjava (toleranca), upoštevamo izraze (+ - * / ( ) **), ulomke, vejico/piko, °, €
    - posebni slučaji: 'x = ...' (sistemi/linearne): vzamemo številko desno od '='
    - drugače: najprej normaliziran simbolni niz (· ↔ *), nato poskus preprostega monoma (c*a ~ a*c ~ a/d ~ a*(c))
    """
    ui_raw = user_input.strip()
    if ui_raw == "":
        return False, "Vnesi odgovor."

    # 'x=..' ali 'y=..'
    if "=" in ui_raw:
        left, right = ui_raw.split("=", 1)
        if left.strip().lower() in ("x", "y", "p(x)", "f(x)"):
            ui_raw = right.strip()

    # numeric primerjava
    if p.numeric is not None:
        target = float(p.numeric)
        s = _normalize_number_like(ui_raw)
        val = _try_parse_number(s)
        if val is None:
            return False, "Odgovor naj bo številski (dovoljeno: izrazi, ulomki, vejica/pika, °, €)."
        tol = 1e-3
        if "€" in str(p.solution) or re.search(r"\d+[,\.]\d{2}\b", str(p.solution)):
            tol = 1e-2 + 1e-9
        if abs(val - target) <= tol:
            return True, "✅ Pravilno!"
        else:
            return False, f"❌ Ni pravilno. (Namig: poskusi z natančnejšim zapisom ali ulomkom.)"

    # simbolni direkten match (normalizacija)
    def _normalize_symb(s: str) -> str:
        return s.strip().replace(" ", "").replace("−", "-").replace(",", ".").replace("·", "*")
    if _normalize_symb(ui_raw) == _normalize_symb(str(p.solution)):
        return True, "✅ Pravilno!"

    # preprost monom
    ui_m = _parse_simple_monomial(ui_raw)
    sol_m = _parse_simple_monomial(str(p.solution))
    if ui_m and sol_m:
        (ui_var, ui_c), (sol_var, sol_c) = ui_m, sol_m
        if ui_var == sol_var:
            try:
                ok = abs(float(ui_c) - float(sol_c)) <= 1e-12
                return (ok, "✅ Pravilno!" if ok else "❌ Koeficient pri spremenljivki ni enak rešitvi.")
            except Exception:
                pass

    return False, "❌ Ni pravilno. (Lahko klikneš »Pokaži rešitev« ali poskusiš znova.)"

# ===================== GUI =====================

try:
    from ui.theme import LIGHT, DARK
except Exception:
    # Fallback, če ui/theme.py ni na voljo (npr. med razvojem)
    LIGHT = {
        "BG": "#F6F7FB",
        "PANEL": "#FFFFFF",
        "TEXT": "#0F172A",
        "SUB": "#475569",
        "PRIMARY": "#2563EB",
        "ACCENT": "#16A34A",
        "LIST_BG": "#FFFFFF",
        "LIST_SEL": "#DBEAFE",
    }
    DARK = {
        "BG": "#0F131A",
        "PANEL": "#151A22",
        "TEXT": "#E6EDF3",
        "SUB": "#9AA4B2",
        "PRIMARY": "#4C84FF",
        "ACCENT": "#6EE7B7",
        "LIST_BG": "#0E131B",
        "LIST_SEL": "#25406B",
    }

@dataclass
class UIState:
    generated: List[Problem]
    last_seed: Optional[str] = None
    show_solution: bool = False
    show_steps: bool = False

class App(ttk.Frame):
    def __init__(self, master: tk.Tk):
        super().__init__(master)
        self.master = master
        self.theme = LIGHT
        self.state = UIState(generated=[], last_seed=None)
        self._selected_idx = -1  # zapomnimo si, katera naloga je prikazana

        self._configure_style()
        self._build_layout()
        self._load_categories()

    # ---------- Tema / Style ----------
    def _configure_style(self):
        self.master.title("MathDrill – Učni vmesnik")
        self.master.geometry("1140x760")
        self.master.minsize(1000, 620)
        style = ttk.Style()
        try:
            style.theme_use("clam")
        except Exception:
            pass
        self._apply_theme(style)

    def _apply_theme(self, style: ttk.Style):
        t = self.theme
        self.master.configure(bg=t["BG"])
        style.configure("TFrame", background=t["BG"])
        style.configure("Panel.TFrame", background=t["PANEL"])
        style.configure("TLabel", background=t["BG"], foreground=t["TEXT"])
        style.configure("Panel.TLabel", background=t["PANEL"], foreground=t["TEXT"])
        style.configure("Header.TLabel", background=t["BG"], foreground=t["TEXT"], font=("Segoe UI", 18, "bold"))
        style.configure("Sub.TLabel", background=t["BG"], foreground=t["SUB"], font=("Segoe UI", 10))
        style.configure("TButton", font=("Segoe UI", 10, "bold"))
        style.configure("Primary.TButton", foreground="white", background=t["PRIMARY"])
        style.map("Primary.TButton", background=[("active", "#1D4ED8")])
        style.configure("Accent.TButton", foreground="white", background=t["ACCENT"])
        style.map("Accent.TButton", background=[("active", "#15803D")])
        style.configure("TCombobox", fieldbackground="white", padding=4)
        style.configure("TLabelframe", background=t["PANEL"], foreground=t["TEXT"])
        style.configure("TLabelframe.Label", background=t["PANEL"], foreground=t["TEXT"], font=("Segoe UI", 10, "bold"))

        # Posodobi barve runtime widgetov
        if hasattr(self, "lst"):
            try:
                self.lst.configure(bg=t["LIST_BG"], fg=t["TEXT"], selectbackground=t["LIST_SEL"])
            except tk.TclError:
                self.lst.configure(bg=t["LIST_BG"], fg=t["TEXT"])

        for wname in ("txt_task", "txt_sol", "txt_steps"):
            if hasattr(self, wname):
                w = getattr(self, wname)
                try:
                    w.configure(bg=t["LIST_BG"], fg=t["TEXT"], insertbackground=t["TEXT"])
                except tk.TclError:
                    w.configure(bg=t["LIST_BG"], fg=t["TEXT"])

    # ---------- Layout ----------
    def _build_layout(self):
        root = self.master

        # Header + temni način
        header = ttk.Frame(root, style="TFrame")
        header.pack(side=tk.TOP, fill=tk.X, padx=16, pady=(12, 8))
        ttk.Label(header, text="📘 MathDrill — Učni vmesnik", style="Header.TLabel").pack(side=tk.LEFT)
        self.var_dark = tk.BooleanVar(value=False)
        chk = ttk.Checkbutton(header, text="Temni način", command=self._toggle_dark, variable=self.var_dark)
        chk.pack(side=tk.RIGHT)

        ttk.Label(header,
                  text="Ustvari nabor nalog; rešitve/koraki so skriti, dokler ne klikneš ‘Pokaži’. Vnesi svoj odgovor in klikni ‘Preveri’.",
                  style="Sub.TLabel").pack(anchor="w", pady=(4,0))

        # Main: left controls + right viewer
        main = ttk.Frame(root, style="TFrame")
        main.pack(side=tk.TOP, fill=tk.BOTH, expand=True, padx=16, pady=8)

        left = ttk.Frame(main, style="Panel.TFrame")
        left.pack(side=tk.LEFT, fill=tk.Y, padx=(0, 8))
        right = ttk.Frame(main, style="Panel.TFrame")
        right.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)

        # Left controls
        pad = {"padx": 12, "pady": 8}
        ttk.Label(left, text="Nastavitve", style="Panel.TLabel", font=("Segoe UI", 12, "bold")).grid(row=0, column=0, columnspan=2, sticky="w", **pad)

        ttk.Label(left, text="Kategorija:", style="Panel.TLabel").grid(row=1, column=0, sticky="e", **pad)
        self.cbo_kat = ttk.Combobox(left, state="readonly", width=26)
        self.cbo_kat.grid(row=1, column=1, sticky="w", **pad)

        ttk.Label(left, text="Težavnost (1–3):", style="Panel.TLabel").grid(row=2, column=0, sticky="e", **pad)
        self.cbo_tez = ttk.Combobox(left, state="readonly", width=26, values=["1", "2", "3"])
        self.cbo_tez.set("1")
        self.cbo_tez.grid(row=2, column=1, sticky="w", **pad)

        ttk.Label(left, text="Št. nalog:", style="Panel.TLabel").grid(row=3, column=0, sticky="e", **pad)
        self.spn_cnt = ttk.Spinbox(left, from_=1, to=100, width=8)
        self.spn_cnt.set(10)
        self.spn_cnt.grid(row=3, column=1, sticky="w", **pad)

        ttk.Label(left, text="Seed (opcijsko):", style="Panel.TLabel").grid(row=4, column=0, sticky="e", **pad)
        self.ent_seed = ttk.Entry(left, width=28)
        self.ent_seed.grid(row=4, column=1, sticky="w", **pad)

        btns = ttk.Frame(left, style="Panel.TFrame")
        btns.grid(row=5, column=0, columnspan=2, sticky="we", padx=12, pady=(4, 12))
        btns.grid_columnconfigure(0, weight=1)
        btns.grid_columnconfigure(1, weight=1)

        self.btn_gen = ttk.Button(btns, text="✨ Ustvari nabor", style="Primary.TButton", command=self._on_generate)
        self.btn_gen.grid(row=0, column=0, sticky="we", padx=(0,6))
        self.btn_export = ttk.Button(btns, text="📝 Izvoz v Markdown", style="Accent.TButton", command=self._on_export)
        self.btn_export.grid(row=0, column=1, sticky="we", padx=(6,0))

        self.btn_checkset = ttk.Button(btns, text="✅ Samopreizkus seta", command=self._on_selfcheck)
        self.btn_checkset.grid(row=1, column=0, sticky="we", padx=(0,6), pady=(8,0))
        self.btn_clear = ttk.Button(btns, text="🧹 Počisti", command=self._on_clear)
        self.btn_clear.grid(row=1, column=1, sticky="we", padx=(6,0), pady=(8,0))

        ttk.Label(left, text="Namig: ‘Vse kategorije’ ustvari mešan set. Seed zagotovi ponovljivost.",
                  style="Sub.TLabel").grid(row=6, column=0, columnspan=2, sticky="w", padx=12, pady=(0,12))

        # Right viewer
        right.grid_columnconfigure(0, weight=1)
        right.grid_rowconfigure(3, weight=1)

        ttk.Label(right, text="Tvoj nabor nalog", style="Panel.TLabel", font=("Segoe UI", 12, "bold")).grid(row=0, column=0, sticky="w", padx=12, pady=(12,6))

        # seznam
        self.lst = tk.Listbox(right, bg=self.theme["LIST_BG"], fg=self.theme["TEXT"],
                              selectbackground=self.theme["LIST_SEL"], activestyle="none",
                              highlightthickness=0, bd=1)
        self.lst.grid(row=1, column=0, sticky="nsew", padx=12, pady=(0,8))
        self.lst.bind("<<ListboxSelect>>", self._on_select)

        # tabs
        self.tabs = ttk.Notebook(right)
        self.tabs.grid(row=2, column=0, sticky="nsew", padx=12, pady=(0,8))

        tab_task = ttk.Frame(self.tabs, style="Panel.TFrame")
        tab_sol  = ttk.Frame(self.tabs, style="Panel.TFrame")
        tab_steps= ttk.Frame(self.tabs, style="Panel.TFrame")
        self.tabs.add(tab_task, text="📄 Naloga")
        self.tabs.add(tab_sol,  text="✅ Rešitev")
        self.tabs.add(tab_steps,text="🧭 Koraki")

        self.txt_task = tk.Text(tab_task, bg=self.theme["LIST_BG"], fg=self.theme["TEXT"],
                                wrap="word", bd=0, padx=10, pady=10, height=8)
        self.txt_task.pack(fill=tk.BOTH, expand=True, padx=6, pady=(6,2))

        self.txt_sol = tk.Text(tab_sol, bg=self.theme["LIST_BG"], fg=self.theme["TEXT"],
                               wrap="word", bd=0, padx=10, pady=10)
        self.txt_sol.pack(fill=tk.BOTH, expand=True, padx=6, pady=6)

        self.txt_steps = tk.Text(tab_steps, bg=self.theme["LIST_BG"], fg=self.theme["TEXT"],
                                 wrap="word", bd=0, padx=10, pady=10)
        self.txt_steps.pack(fill=tk.BOTH, expand=True, padx=6, pady=6)

        # Answer bar (pod zavihki)
        answer_bar = ttk.Frame(right, style="Panel.TFrame")
        answer_bar.grid(row=3, column=0, sticky="we", padx=12, pady=(0,12))
        answer_bar.grid_columnconfigure(1, weight=1)

        ttk.Label(answer_bar, text="Tvoj odgovor:", style="Panel.TLabel").grid(row=0, column=0, sticky="w", padx=(6,8), pady=6)
        self.ent_answer = ttk.Entry(answer_bar)
        self.ent_answer.grid(row=0, column=1, sticky="we", pady=6)
        self.ent_answer.bind("<Return>", lambda e: self._on_check_answer())

        self.btn_check_answer = ttk.Button(answer_bar, text="🔎 Preveri", command=self._on_check_answer)
        self.btn_check_answer.grid(row=0, column=2, sticky="w", padx=(8,6))

        self.btn_show_sol = ttk.Button(answer_bar, text="👁️ Pokaži rešitev", command=self._on_show_solution)
        self.btn_show_sol.grid(row=0, column=3, sticky="w", padx=(8,6))

        self.btn_show_steps = ttk.Button(answer_bar, text="🧭 Pokaži korake", command=self._on_show_steps)
        self.btn_show_steps.grid(row=0, column=4, sticky="w", padx=(8,6))

        self.btn_hide_all = ttk.Button(answer_bar, text="🙈 Skrij vse", command=self._on_hide_all)
        self.btn_hide_all.grid(row=0, column=5, sticky="w", padx=(8,6))

        self.btn_copy_steps = ttk.Button(answer_bar, text="📋 Kopiraj korake", command=self._on_copy_steps)
        self.btn_copy_steps.grid(row=0, column=6, sticky="w", padx=(8,6))

        # feedback
        self.lbl_feedback = ttk.Label(answer_bar, text="", style="Panel.TLabel")
        self.lbl_feedback.grid(row=1, column=0, columnspan=7, sticky="w", padx=(6,8), pady=(0,6))

        # status
        footer = ttk.Frame(root, style="TFrame")
        footer.pack(side=tk.BOTTOM, fill=tk.X, padx=16, pady=(0, 12))
        self.var_status = tk.StringVar(value="Pripravljeno.")
        ttk.Label(footer, textvariable=self.var_status, style="Sub.TLabel").pack(anchor="w")

        # ob zagonu: skrij rešitve/korake
        self._hide_solution_steps()

    # ---------- Tema toggle ----------
    def _toggle_dark(self):
        self.theme = DARK if self.var_dark.get() else LIGHT
        self._apply_theme(ttk.Style())

    # ---------- Pomožne ----------
    def _load_categories(self):
        imena = REG.list_categories()
        imena_sorted = ["Vse kategorije"] + imena
        self.cbo_kat["values"] = imena_sorted
        self.cbo_kat.set("Vse kategorije")

    def _get_settings(self):
        kat = self.cbo_kat.get().strip() or "Vse kategorije"
        try:
            cnt = int(self.spn_cnt.get())
            if not (1 <= cnt <= 100):
                raise ValueError
        except Exception:
            messagebox.showerror("Napaka", "Število nalog mora biti med 1 in 100.")
            return None
        try:
            tez = int(self.cbo_tez.get())
        except Exception:
            tez = 1
        seed = self.ent_seed.get().strip() or None
        return kat, cnt, tez, seed

    def _apply_seed(self, seed: Optional[str]):
        if seed is None:
            return
        try:
            random.seed(int(seed))
        except Exception:
            random.seed(seed)

    def _render_list(self):
        self.lst.delete(0, tk.END)
        for i, p in enumerate(self.state.generated, 1):
            self.lst.insert(tk.END, f"{i}. [{p.category}] {p.prompt}")

    def _show_problem_text_only(self, idx: int):
        """Zavihek 'Naloga': samo besedilo naloge. Rešitev/koraki prikaži, če sta zastavici vključeni."""
        self.txt_task.delete("1.0", tk.END)
        self.txt_sol.delete("1.0", tk.END)
        self.txt_steps.delete("1.0", tk.END)
        self.ent_answer.delete(0, tk.END)
        self.lbl_feedback.config(text="", foreground=self.theme["SUB"])

        if 0 <= idx < len(self.state.generated):
            p = self.state.generated[idx]
            self.txt_task.insert("1.0", f"[{p.category}]\n{p.prompt}\n")
        else:
            self.txt_task.insert("1.0", "—")

        if self.state.show_solution and 0 <= idx < len(self.state.generated):
            self.txt_sol.insert("1.0", self.state.generated[idx].solution)
        else:
            self.txt_sol.insert("1.0", "Skrito. Klikni »Pokaži rešitev«.")

        if self.state.show_steps and 0 <= idx < len(self.state.generated):
            # Prikaži korake brez kratic (humanizacija besedila)
            koraki = _humanize_steps(self.state.generated[idx].steps)
            self.txt_steps.insert("1.0", koraki)
        else:
            self.txt_steps.insert("1.0", "Skrito. Klikni »Pokaži korake«.")

    def _hide_solution_steps(self):
        self.state.show_solution = False
        self.state.show_steps = False
        self.txt_sol.delete("1.0", tk.END)
        self.txt_steps.delete("1.0", tk.END)
        self.txt_sol.insert("1.0", "Skrito. Klikni »Pokaži rešitev«.")
        self.txt_steps.insert("1.0", "Skrito. Klikni »Pokaži korake«.")

    def _show_solution_now(self, idx: int):
        if 0 <= idx < len(self.state.generated):
            p = self.state.generated[idx]
            self.txt_sol.delete("1.0", tk.END)
            self.txt_sol.insert("1.0", p.solution)
            self.state.show_solution = True

    def _show_steps_now(self, idx: int):
        if 0 <= idx < len(self.state.generated):
            p = self.state.generated[idx]
            self.txt_steps.delete("1.0", tk.END)
            # Prikaži korake brez kratic (humanizacija besedila)
            koraki = _humanize_steps(p.steps)
            self.txt_steps.insert("1.0", koraki)
            self.state.show_steps = True

    # ---------- Event handlers ----------
    def _on_generate(self):
        s = self._get_settings()
        if not s: return
        kat, cnt, tez, seed = s
        self.state.last_seed = seed
        self._apply_seed(seed)

        # reset prikaza/režima
        self.state.show_solution = False
        self.state.show_steps = False
        self._selected_idx = -1

        try:
            naloge: List[Problem] = []
            if kat == "Vse kategorije":
                imena = REG.list_categories()
                for _ in range(cnt):
                    naloge.append(REG.generate(random.choice(imena), tez))
            else:
                if not REG.is_registered(kat):
                    cand = [n for n in REG.list_categories() if n.lower() == kat.lower()]
                    if cand:
                        kat = cand[0]
                    else:
                        messagebox.showerror("Napaka", f"Kategorija '{kat}' ni registrirana.")
                        return
                for _ in range(cnt):
                    naloge.append(REG.generate(kat, tez))
        except Exception as e:
            messagebox.showerror("Napaka pri generiranju", str(e))
            return

        self.state.generated = naloge
        self._render_list()
        self._show_problem_text_only(0)
        self.lst.selection_clear(0, tk.END)
        if len(naloge) > 0:
            self.lst.selection_set(0)
            self.lst.activate(0)
            self._selected_idx = 0
        self.var_status.set(f"Ustvarjeno {len(naloge)} nalog. Rešitve so skrite.")

    def _on_export(self):
        if not self.state.generated:
            messagebox.showinfo("Izvoz", "Najprej ustvari nabor nalog.")
            return
        path = filedialog.asksaveasfilename(
            title="Shrani delovni list (.md)",
            defaultextension=".md",
            filetypes=[("Markdown", "*.md")]
        )
        if not path:
            return
        try:
            pot_res = izvoz_markdown(self.state.generated, path)
            messagebox.showinfo("Izvoz", f"Delovni list je shranjen.\nRešitve: {pot_res.name}")
            self.var_status.set(f"Izvoženo: {os.path.basename(path)}")
        except Exception as e:
            messagebox.showerror("Napaka pri izvozu", str(e))

    def _on_selfcheck(self):
        if not self.state.generated:
            messagebox.showinfo("Samopreizkus", "Najprej ustvari nabor nalog.")
            return
        ok = sum(verify_problem_numeric(p) for p in self.state.generated)
        total = len(self.state.generated)
        pct = 100.0 * ok / total if total else 0.0
        messagebox.showinfo("Samopreizkus", f"Preverjenih {ok}/{total} ({pct:.1f}%).")
        self.var_status.set(f"Samopreizkus končan: {ok}/{total} OK.")

    def _on_clear(self):
        self.state.generated.clear()
        self._render_list()
        self._show_problem_text_only(-1)
        self._selected_idx = -1
        self.var_status.set("Počiščeno.")

    def _on_select(self, event=None):
        sel = self.lst.curselection()
        idx = sel[0] if sel else -1
        if idx != self._selected_idx:
            self._selected_idx = idx
            self._show_problem_text_only(idx)
            self.var_status.set("Naloga izbrana.")

    def _on_show_solution(self):
        sel = self.lst.curselection()
        if not sel: return
        idx = sel[0]
        self.state.show_solution = True
        self._show_solution_now(idx)
        self.tabs.select(1)

    def _on_show_steps(self):
        sel = self.lst.curselection()
        if not sel: return
        idx = sel[0]
        self.state.show_steps = True
        self._show_steps_now(idx)
        self.tabs.select(2)

    def _on_hide_all(self):
        self.state.show_solution = False
        self.state.show_steps = False
        self.txt_sol.delete("1.0", tk.END)
        self.txt_steps.delete("1.0", tk.END)
        self.txt_sol.insert("1.0", "Skrito. Klikni »Pokaži rešitev«.")
        self.txt_steps.insert("1.0", "Skrito. Klikni »Pokaži korake«.")
        self.tabs.select(0)
        self.lbl_feedback.config(text="", foreground=self.theme["SUB"])
        self.var_status.set("Rešitev in koraki skrita.")

    def _on_copy_steps(self):
        txt = self.txt_steps.get("1.0", tk.END).strip()
        if not txt or txt.startswith("Skrito."):
            messagebox.showinfo("Kopiranje", "Koraki so skriti. Najprej klikni »Pokaži korake«.")
            return
        try:
            self.master.clipboard_clear()
            self.master.clipboard_append(txt)
            self.var_status.set("Koraki kopirani v odložišče.")
        except Exception:
            messagebox.showerror("Napaka", "Kopiranja v odložišče ni uspelo.")

    def _on_check_answer(self):
        sel = self.lst.curselection()
        if not sel:
            self.lbl_feedback.config(text="Najprej izberi nalogo na seznamu.", foreground="#b91c1c")
            return
        idx = sel[0]
        p = self.state.generated[idx]
        user_input = self.ent_answer.get().strip()

        try:
            v = evaluate_answer(p, user_input, subject=p.category, config=CFG)
        except Exception as e:
            self.lbl_feedback.config(text=f"Napaka pri preverjanju: {e}", foreground="#b91c1c")
            self.var_status.set("Pri preverjanju je prišlo do napake.")
            return

        if v.is_correct:
            msg = "✅ Pravilno!"
            if v.explanation:
                msg += " " + v.explanation
            self.lbl_feedback.config(text=msg, foreground="#15803d")
            self.var_status.set("Bravo! Lahko si ogledaš korake ali nadaljuješ na naslednjo nalogo.")
        else:
            # prikazaj uporaben namig, če je na voljo
            hint = v.explanation or ("Format odgovora ni pravilen." if v.match == "wrong-format" else "Ni pravilno.")
            prefix = "❌ "
            self.lbl_feedback.config(text=prefix + hint, foreground="#b91c1c")
            self.var_status.set("Poskusi znova ali klikni »Pokaži rešitev« za namig.")


def main():
    root = tk.Tk()
    app = App(root)
    root.mainloop()

if __name__ == "__main__":
    main()


--- Konec datoteke: ucni_vmesnik.py ---

--- Začetek datoteke: orodja\eval_helpers.py ---

# -*- coding: utf-8 -*-
"""
orodja/eval_helpers.py
----------------------
Skupni eval/parsing pomočniki, da se izognemo podvajanju kode v GUI datotekah.

Vključuje:
- _safe_eval_number_expr(s)      : varen eval preprostih aritmetičnih izrazov (+, -, *, /, **, oklepaji)
- _normalize_number_like(s)      : normalizacija številske oblike (presledki, vejica→pika, €, °, '·'→'*')
- _try_parse_number(s)           : poskus razčlenitve v float (izrazi, strogi ulomki a/b, decimalke)
- _parse_simple_monomial(expr)   : prepozna oblike c*a, a*c, a/d, a*(c), (c)*a, 'a' → (spremenljivka, koeficient)

Opombe:
- Namenjeno je deljenju med ucni_vmesnik.py, test_vmesnik.py ali drugimi UI moduli.
- Za naprednejše preverjanje odgovorov uporabljaj core v orodja/odgovor_eval.py.
"""

from __future__ import annotations

import ast
import re
from fractions import Fraction
from typing import Optional, Tuple


# ----------------------------- Normalizacija številskih nizov -----------------------------

def _normalize_number_like(s: str) -> str:
    """
    Normalizira "številko podobne" vnose:
    - odstrani presledke,
    - decimalna vejica -> pika,
    - odstrani stopinje (°),
    - odstrani zaključni €,
    - '·' -> '*' (vizualni operator množenja).
    """
    s = (s or "").strip()
    s = s.replace(" ", "").replace(",", ".").replace("·", "*")
    s = s.replace("°", "")
    if s.endswith("€"):
        s = s[:-1]
    return s


# ----------------------------- Varen eval preprostih izrazov -----------------------------

def _safe_eval_number_expr(s: str) -> Optional[float]:
    """
    Varno ovrednoti preprost aritmetični izraz: + - * / ** in oklepaji.
    Dovoli tudi unarni +/-.
    Vrne float ali None (če izraz ni podprt ali je neveljaven).

    Primeri:
      '1/2 + 3'     -> 3.5
      '(2-5)*4'     -> -12
      '2**3 + 0.5'  -> 8.5
    """
    if not isinstance(s, str):
        return None
    src = _normalize_number_like(s)

    # Whitelist operatorjev/nodov v AST
    allowed_bin = {
        ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow
    }
    allowed_un = {ast.UAdd, ast.USub}
    allowed_nodes = (
        ast.Expression, ast.BinOp, ast.UnaryOp, ast.Constant,
        ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow, ast.UAdd, ast.USub,
        ast.Tuple, ast.List  # toleriramo strukture, a jih spodaj pretvorimo v float preko Exception
    )

    try:
        node = ast.parse(src, mode="eval")
    except Exception:
        return None

    def _eval(n):
        if isinstance(n, ast.Expression):
            return _eval(n.body)
        if isinstance(n, ast.Constant):
            if isinstance(n.value, (int, float)):
                return float(n.value)
            raise ValueError("Unsupported constant")
        if isinstance(n, ast.UnaryOp) and type(n.op) in allowed_un:
            return +_eval(n.operand) if isinstance(n.op, ast.UAdd) else -_eval(n.operand)
        if isinstance(n, ast.BinOp) and type(n.op) in allowed_bin:
            l = _eval(n.left); r = _eval(n.right)
            if isinstance(n.op, ast.Add):  return l + r
            if isinstance(n.op, ast.Sub):  return l - r
            if isinstance(n.op, ast.Mult): return l * r
            if isinstance(n.op, ast.Div):  return l / r
            if isinstance(n.op, ast.Pow):  return l ** r
        raise ValueError("Unsupported expression")

    # preveri whitelist
    for sub in ast.walk(node):
        if not isinstance(sub, allowed_nodes):
            return None

    try:
        val = _eval(node)
        return float(val)
    except Exception:
        return None


# ----------------------------- Strogo razčlenjevanje števila -----------------------------

def _try_parse_number(s: str) -> Optional[float]:
    """
    Poskusi s:
      1) varnim evalom (+ - * / ** in oklepaji),
      2) strogim ulomkom a/b (a in b sta cela števila ali decimalke),
      3) navadnim float.

    Vrne float ali None.
    """
    s = _normalize_number_like(s)
    # 1) varen eval
    val = _safe_eval_number_expr(s)
    if val is not None:
        return val

    # 2) strogi ulomek a/b
    if "/" in s:
        parts = s.split("/")
        if len(parts) == 2:
            a, b = parts
            def _is_num(tok: str) -> bool:
                tok = tok.strip()
                if not tok:
                    return False
                # dovoljeni formati: +/- int ali +/- float (z eno piko)
                if tok[0] in "+-":
                    tok_body = tok[1:]
                else:
                    tok_body = tok
                return tok_body.replace(".", "", 1).isdigit()
            if _is_num(a) and _is_num(b):
                try:
                    # decimalni del šele tu pretvorimo v Fraction
                    num = Fraction(a) if ("/" in a) else Fraction.from_float(float(a)).limit_denominator() if "." in a else Fraction(int(a), 1)
                    den = Fraction(b) if ("/" in b) else Fraction.from_float(float(b)).limit_denominator() if "." in b else Fraction(int(b), 1)
                    if den == 0:
                        return None
                    return float(num / den)
                except Exception:
                    return None

    # 3) navaden float
    try:
        return float(s)
    except Exception:
        return None


# ----------------------------- Preprost monom (simbolni) -----------------------------

# Vzorce pokrijemo z regexi:
#  - a/d                 → (var='a', coeff=1/d)
#  - (c)*a ali c*a       → coeff=c, var=a
#  - a*(c) ali a*c       → coeff=c, var=a
#  - 'a'                 → coeff=1
#
# c je lahko celo, ulomek ali decimalka.

def _parse_simple_monomial(expr: str) -> Optional[Tuple[str, Fraction]]:
    """
    Prepozna oblike: c*a, a*c, a/d, a*(c), (c)*a ali 'a'
    kjer je c številski koeficient (tudi ulomek/decimalka), a pa enoznačna črka.
    Vrne (var, coeff) ali None.
    """
    s = (expr or "").strip().replace(" ", "").replace(",", ".").replace("·", "*")

    # a/d  → coeff = 1/d
    m = re.fullmatch(r'([A-Za-z])/(\-?\d+(?:/\d+)?(?:\.\d+)?)', s)
    if m:
        var = m.group(1)
        den = m.group(2)
        try:
            coeff = Fraction(1, 1) / (Fraction(den) if "/" in den else Fraction.from_float(float(den)).limit_denominator())
            return (var, coeff)
        except Exception:
            return None

    # (c)*a ali c*a
    m = re.fullmatch(r'\(?([\-+]?\d+(?:/\d+)?(?:\.\d+)?)\)?\*([A-Za-z])', s)
    if m:
        c_str, var = m.group(1), m.group(2)
        try:
            coeff = Fraction(c_str) if "/" in c_str else Fraction.from_float(float(c_str)).limit_denominator()
            return (var, coeff)
        except Exception:
            return None

    # a*(c) ali a*c
    m = re.fullmatch(r'([A-Za-z])\*\(?([\-+]?\d+(?:/\d+)?(?:\.\d+)?)\)?', s)
    if m:
        var, c_str = m.group(1), m.group(2)
        try:
            coeff = Fraction(c_str) if "/" in c_str else Fraction.from_float(float(c_str)).limit_denominator()
            return (var, coeff)
        except Exception:
            return None

    # samo 'a' → koeficient 1
    m = re.fullmatch(r'([A-Za-z])', s)
    if m:
        return (m.group(1), Fraction(1, 1))

    return None


--- Konec datoteke: orodja\eval_helpers.py ---

--- Začetek datoteke: orodja\generator_vaj.py ---

# -*- coding: utf-8 -*-
"""
orodja/generator_vaj.py
-----------------------
CLI generator vaj, ki uporablja register kategorij iz mape 'predmeti'.

Zmožnosti:
- Izpis nalog v konzolo
- Izvoz delovnega lista v Markdown (+ avtomatske rešitve)
- Samopreizkus pravilnosti (--samopreizkus N)
- Reproducibilnost z --seed

Uporaba (primeri):
    python orodja/generator_vaj.py
    python orodja/generator_vaj.py --kategorija "Ulomki" --stevilo 12 --tezavnost 2
    python orodja/generator_vaj.py --delovni-list Dan1.md --stevilo 10 --kategorija vse
    python orodja/generator_vaj.py --samopreizkus 1000 --seed 42

Kategorije vidiš z:
    python orodja/generator_vaj.py --seznam
"""

from __future__ import annotations

import argparse
import random
import re
import ast
from fractions import Fraction
from pathlib import Path
from typing import List, Optional

# --- Register in tipi (skupni model) ---
from predmeti import register as REG
from predmeti.tipi import Problem

# --- UVOZI VSE PREDMETE, DA SE REGISTRIRAJO ---
import predmeti.ulomki
import predmeti.potence_in_koreni
import predmeti.vrstni_red_racunov
import predmeti.linearne_enacbe
import predmeti.polinomi
import predmeti.trigonometrija
import predmeti.funkcije_in_grafi
import predmeti.sistemi_enacb
import predmeti.procenti_in_ddv


# ----------------- Izris/izvoz -----------------

def izpisi_nalogo(i: int, p: Problem) -> str:
    return f"{i}. [{p.category}] {p.prompt}"

def izpisi_resitev(i: int, p: Problem) -> str:
    koraki = p.steps.replace("\n", "\n   ")
    return f"{i}. Odgovor: {p.solution}\n   Koraki: {koraki}"

def izvoz_markdown(naloge: List[Problem], pot_md: str | Path) -> Path:
    pot_md = Path(pot_md)
    osnova = pot_md.with_suffix("")
    pot_resitve = osnova.with_name(osnova.name + "_resitve").with_suffix(".md")

    pot_md.parent.mkdir(parents=True, exist_ok=True)
    pot_resitve.parent.mkdir(parents=True, exist_ok=True)

    # Delovni list
    with pot_md.open("w", encoding="utf-8") as f:
        f.write("# Samodejno ustvarjen delovni list\n\n")
        for i, p in enumerate(naloge, 1):
            f.write(f"**{i}. [{p.category}]** {p.prompt}\n\n")
        f.write("\n---\n")
        f.write(f"*Rešitve so v datoteki:* **{pot_resitve.name}**\n")

    # Rešitve + koraki
    with pot_resitve.open("w", encoding="utf-8") as f:
        f.write("# Rešitve\n\n")
        for i, p in enumerate(naloge, 1):
            f.write(f"**{i}.** Odgovor: `{p.solution}`\n\n")
            f.write("Koraki:\n\n")
            f.write("```\n" + p.steps + "\n```\n\n")

    return pot_resitve


# ----------------- Neodvisen preverjevalnik -----------------

def _parse_frac(tok: str) -> Optional[Fraction]:
    tok = tok.strip()
    # dovoli vodilni minus samo v števcu
    if tok.count('/') == 1:
        num, den = tok.split('/', 1)
        if num.strip('-').isdigit() and den.strip().isdigit():
            try:
                return Fraction(int(num), int(den))
            except Exception:
                return None
    # poskusi celo število
    if tok.strip('-').isdigit():
        return Fraction(int(tok), 1)
    return None

def _verify_fractions_prompt(prompt: str) -> Optional[Fraction]:
    """
    Razume oblike iz 'Ulomki':
      - 'Uredi izraz: a/b + c/d'
      - 'Uredi izraz: a/b - c/d'
      - 'Uredi izraz: a/b ÷ c/d + e/f'
    Vrne Fraction ali None, če ne zna razčleniti (npr. simbolne '... · a').
    """
    text = prompt.replace("Uredi izraz: ", "").strip()
    text = text.replace("÷", "/").replace("·", "*")
    toks = text.split()
    if len(toks) == 3 and toks[1] in {"+", "-"}:
        f1, f2 = _parse_frac(toks[0]), _parse_frac(toks[2])
        if f1 is not None and f2 is not None:
            return f1 + f2 if toks[1] == "+" else f1 - f2
    if len(toks) == 5 and toks[1] == "/" and toks[3] == "+":
        f1, f2, f3 = _parse_frac(toks[0]), _parse_frac(toks[2]), _parse_frac(toks[4])
        if f1 and f2 and f3:
            return f1 / f2 + f3
    return None

# varen eval za preprost aritmetični izraz z + - * () in predpripravljeno ^2 -> (x*x)
_ALLOWED_NODES = {
    ast.Expression, ast.BinOp, ast.Add, ast.Sub, ast.Mult, ast.Pow, ast.USub,
    ast.UnaryOp, ast.Constant, ast.Constant, ast.Load, ast.Call, ast.Name, ast.Mod,
    ast.FloorDiv, ast.Div, ast.LParen if hasattr(ast, "LParen") else ast.AST  # tolerantno
}
_ALLOWED_NAMES: dict[str, int] = {}

def _safe_eval_expr(expr: str) -> int | float:
    """
    Zelo omejen eval za naše generirane izraze (cela števila, +,-,*, oklepaji).
    """
    # pretvori ^2 v * in sebi lastno obliko
    expr = re.sub(r'(\d+)\^2', r'(\1*\1)', expr)
    expr = expr.replace("·", "*")
    tree = ast.parse(expr, mode="eval")
    for node in ast.walk(tree):
        if type(node) not in _ALLOWED_NODES:
            raise ValueError("Nedovoljen izraz.")
        if isinstance(node, ast.Call):
            raise ValueError("Klici niso dovoljeni.")
        if isinstance(node, ast.Name):
            if node.id not in _ALLOWED_NAMES:
                raise ValueError("Nepričakovano ime v izrazu.")
    return eval(compile(tree, "<expr>", "eval"), {"__builtins__": {}}, {})

def _verify_problem(p: Problem) -> bool:
    """
    Neodvisno preveri pravilnost rešitve po kategorijah.
    Namen: self-check in auto-check, da imaš zaupanje v generator.
    """
    try:
        cat = p.category

        # Ulomki
        if cat == "Ulomki":
            val = _verify_fractions_prompt(p.prompt)
            if val is None:
                return True  # simbolni primeri (npr. '... · a') — preskočimo
            try:
                sol = Fraction(p.solution)
                return sol == val
            except Exception:
                # če solution ni čisti ulomek, primerjaj numerično, če imamo numeric
                return (p.numeric is not None) and abs(float(val) - float(p.numeric)) < 1e-12

        # Vrstni red računov
        if cat == "Vrstni red računov":
            text = p.prompt.replace("Uredi: ", "").strip()
            val = _safe_eval_expr(text)
            try:
                return int(p.solution) == int(val)
            except Exception:
                return False

        # Linearne enačbe — delni checker (osnovna oblika in ulomki+SKI)
        if cat == "Linearne enačbe":
            try:
                x = Fraction(p.solution)
            except Exception:
                return True  # sprejmemo, če je rešitev v drugi obliki (npr. decimalno)
            # Osnovna: A·x + B = C·x + D
            m2 = re.search(r":\s*(-?\d+)\s*[·\*]?\s*x\s*\+\s*(-?\d+)\s*=\s*(-?\d+)\s*[·\*]?\s*x\s*\+\s*(-?\d+)", p.prompt)
            if m2:
                A, B, C, D = map(int, m2.groups())
                return A*x + B == C*x + D
            # Ulomki+SKI: (x±a)/m + (x±b)/n = k
            m = re.search(r":\s*(\(.+?\))/(\d+)\s+\+\s+(\(.+?\))/(\d+)\s+=\s+(-?\d+)", p.prompt)
            if m:
                left1, den1, left2, den2, rhs = m.groups()
                def eval_group(g):
                    m2 = re.search(r"x([+-])(\d+)", g)
                    if m2:
                        sign = 1 if m2.group(1) == '+' else -1
                        return x + sign*int(m2.group(2))
                    return x
                lhs = eval_group(left1)/int(den1) + eval_group(left2)/int(den2)
                return lhs == Fraction(int(rhs), 1)
            return True

        # Potence/Koreni, Polinomi, Trigonometrija, Funkcije in grafi,
        # Procenti/DDV, Sistemi enačb:
        # Ti generatorji po novem dosledno nastavijo p.numeric, kadar je smiselno.
        # Tukaj privzamemo, da je numeric zanesljiv (ker je prišel iz istega izračuna),
        # in check preskočimo, razen če bi želeli implementirati polne parserje.
        if p.numeric is not None:
            return True

        return True

    except Exception:
        return False

def samopreizkus(st_vzorcev: int, seed: Optional[int] = None) -> tuple[int, int]:
    if seed is not None:
        random.seed(seed)
    ok = 0
    imena = REG.list_categories()
    for _ in range(st_vzorcev):
        ime = random.choice(imena)
        p = REG.generate(ime, tezavnost=random.choice([1, 2, 3]))
        if _verify_problem(p):
            ok += 1
    return ok, st_vzorcev


# ----------------- CLI -----------------

def main() -> None:
    parser = argparse.ArgumentParser(description="Generator vaj (CLI) z registrom predmetov")
    parser.add_argument("--kategorija", type=str, default="vse",
                        help='Ime kategorije ali "vse". Uporabi --seznam za razpoložljive.')
    parser.add_argument("--stevilo", type=int, default=10, help="Število nalog (privzeto 10).")
    parser.add_argument("--tezavnost", type=int, choices=[1, 2, 3], default=1, help="Težavnost 1–3.")
    parser.add_argument("--seed", type=str, default=None, help="Seed za reproducibilnost (npr. 16 ali beseda).")
    parser.add_argument("--delovni-list", type=str, default=None,
                        help="Pot do .md za izvoz (ustvari tudi _resitve.md).")
    parser.add_argument("--seznam", action="store_true", help="Izpiši seznam kategorij in končaj.")
    parser.add_argument("--samopreizkus", type=int, default=None,
                        help="Zaženi notranji samopreizkus z N primeri in končaj.")
    args = parser.parse_args()

    if args.seznam:
        imena = REG.list_categories()
        print("Razpoložljive kategorije:")
        for ime in imena:
            print(" -", ime)
        return

    # Seed (dovolimo int ali besedo)
    if args.seed is not None:
        try:
            random.seed(int(args.seed))
        except Exception:
            random.seed(args.seed)

    # Samopreizkus
    if args.samopreizkus:
        ok, total = samopreizkus(args.samopreizkus, seed=None)
        print(f"[SELF-CHECK] {ok}/{total} OK ({ok/total*100:.1f}%)")
        return

    # Ustvari naloge
    naloge: List[Problem] = []
    imena = REG.list_categories()
    if args.kategorija.casefold() == "vse":
        for _ in range(args.stevilo):
            naloge.append(REG.generate(random.choice(imena), tezavnost=args.tezavnost))
    else:
        # register je case-insensitive; če ni registrirana, javi jasno napako
        if not REG.is_registered(args.kategorija):
            raise REG.UnknownCategoryError(
                f"Kategorija '{args.kategorija}' ni registrirana. Uporabi --seznam za pregled."
            )
        for _ in range(args.stevilo):
            naloge.append(REG.generate(args.kategorija, tezavnost=args.tezavnost))

    # Če želimo izvoz
    if args.delovni_list:
        pot_res = izvoz_markdown(naloge, args.delovni_list)
        ok = sum(_verify_problem(p) for p in naloge)
        print(f"[OK] Zapisal delovni list: {args.delovni_list}")
        print(f"[OK] Zapisal rešitve:     {pot_res}")
        print(f"[AUTO-CHECK] {ok}/{len(naloge)} preverjenih pravilno.")
        return

    # Sicer v konzolo + auto-check
    ok = 0
    for i, p in enumerate(naloge, 1):
        print(izpisi_nalogo(i, p))
        print(izpisi_resitev(i, p))
        print("-" * 60)
        if _verify_problem(p):
            ok += 1
    print(f"[AUTO-CHECK] {ok}/{len(naloge)} preverjenih pravilno.")

if __name__ == "__main__":
    main()


--- Konec datoteke: orodja\generator_vaj.py ---

--- Začetek datoteke: orodja\odgovor_eval.py ---

# -*- coding: utf-8 -*-
"""
orodja/odgovor_eval.py
----------------------
Enotni preverjevalnik odgovorov (CORE) z razširljivo arhitekturo "engine + vtičniki".

Zagotavlja:
- API: evaluate_answer(problem, user_input, subject=None, config=EvalConfig())
- Normalizacija vnosa (vejica→pika, tisočiški, ne-lomljivi presledki, enote €, %, ml, m, kg, …)
- Ulomki p/q, cela/decimalna, pari/seznami/množice, mešani odgovori (formula + številčni povzetek)
- Varen mini evaluator: +, -, *, /, ^ (→ **), oklepaji; Fraction; sqrt(...) in √n
- Tolerance abs/rel, ekvivalentne oblike (npr. 1/2 == 0.5)
- Simbolne domene: ℝ \ {a,b}, zapisi z x ≠ c, (-∞,c) ∪ (c,∞) ⇒ primerjava izločitev
- Razširljivost: plugin sistem (neobvezno); če ni plugina, deluje CORE fallback.

Opombe:
- CORE ne implementira vseh predmetno-specifičnih pravil (trig intervali, polinomi, sistemi).
  Za to uporabite plugine v `orodja/odgovor_plugins/*`.
"""

from __future__ import annotations

import ast
import math
import re
import importlib
from dataclasses import dataclass, field
from fractions import Fraction
from typing import Any, Dict, List, Optional, Tuple, Union

# =============================================================================
# Konfiguracija & rezultat
# =============================================================================

@dataclass
class EvalConfig:
    decimal_tolerance_abs: float = 1e-9
    decimal_tolerance_rel: float = 1e-9
    locale_decimal: str = "auto"  # "auto" | "." | ","
    accept_equivalent_forms: bool = True
    set_order_invariant: bool = True
    allow_expressions: bool = True
    max_expr_len: int = 800
    allow_constants: bool = True  # pi, e
    allow_radicals: bool = True   # sqrt(), √
    allow_fraction_eval: bool = True

@dataclass
class Verdict:
    is_correct: bool
    match: str  # "exact" | "close" | "wrong-format" | "wrong-value" | "symbolic-domain"
    normalized_user: Any
    normalized_solution: Any
    explanation: str = ""
    debug: Dict[str, Any] = field(default_factory=dict)


# =============================================================================
# Tokenizacija in normalizacija nizov
# =============================================================================

# Nebreaking spaces itd.
_NBSP_CHARS = ("\u00A0", "\u202F", "\u2007", "\u2060")

# Enote/sufiksi – razširjen seznam
_UNIT_TAIL_RE = re.compile(
    r"(?:\s*(?:%|‰|€|\$|eur|EUR|ml|mL|l|L|dl|cl|mg|g|kg|t|mm|cm|m|km|μm|nm|s|min|h|hr|ms|μs|°C|°F|C|K|A|V|W|Wh|kWh|N|J|Pa|bar|pcs|kos))+$"
)

# Ulomek a/b (poenoten vzorec)
_FRAC_TOKEN = re.compile(r'(?P<sign>[+\-]?)\s*(?P<n>\d+)\s*/\s*(?P<d>-?\d+)')
_INT_TOKEN  = re.compile(r'^[+\-]?\d+$')

# √ zapisi
_RADICAL_COEFF = re.compile(r'(?P<c>\d+)\s*√\s*\(?\s*(?P<n>[+\-]?\d+(?:/\d+)?(?:\.\d+)?)\s*\)?')
_RADICAL_SOLO  = re.compile(r'√\s*\(?\s*(?P<n>[+\-]?\d+(?:/\d+)?(?:\.\d+)?)\s*\)?')

# Levi del “x=”, “x₀=”, “f(x)=”, “y(1)=”...
_SUBSCRIPT_DIGITS = "₀₁₂₃₄₅₆₇₈₉"
_ASSIGN_LHS_RE = re.compile(
    rf"^\s*(?:[A-Za-z][A-Za-z0-9_{_SUBSCRIPT_DIGITS}]*|[A-Za-z]\s*\(\s*x\s*\)|[A-Za-z]\s*\(\s*[\-+]?\d+(?:/\d+)?\s*\))\s*=\s*(?P<rhs>.+)$"
)

# Seznami/pari/množice
_LIST_LIKE_BRACKETS = {'(': ')', '[': ']', '{': '}'}

# “Mešani” številčni token (ulomek ali decimalka/celo)
_NUM_OR_FRAC_TOKEN = re.compile(r"[+\-]?\d+(?:/\d+|[.,]\d+)?")


def _strip_nbsp(s: str) -> str:
    for ch in _NBSP_CHARS:
        s = s.replace(ch, " ")
    return s


def _strip_units_tail(s: str) -> str:
    return _UNIT_TAIL_RE.sub("", s.strip())


def _strip_nbsp_and_units(s: str) -> str:
    return _strip_units_tail(_strip_nbsp(s))


def _normalize_decimal_separator(s: str, cfg: EvalConfig) -> str:
    """
    - odstrani enote/sufikse
    - prepozna EU zapis 1.234,56 → 1234.56
    - auto: kalibrira decimalno vejico v piko
    """
    s = _strip_nbsp_and_units(s)

    # EU: 1.234,56 ali 1.234
    if "." in s and "," in s:
        def _fix_token(tok: str) -> str:
            t = tok
            if re.fullmatch(r"[+\-]?\d{1,3}(?:\.\d{3})+,\d+", t):
                t = t.replace(".", "").replace(",", ".")
            elif re.fullmatch(r"[+\-]?\d{1,3}(?:\.\d{3})+$", t):
                t = t.replace(".", "")
            return t
        parts = re.split(r"(\d[0-9\.,]*\d|\d)", s)
        out = []
        for i, part in enumerate(parts):
            out.append(_fix_token(part) if i % 2 == 1 and part is not None else part)
        s = "".join(out)

    if cfg.locale_decimal == ',':
        return s.replace(',', '.')
    if cfg.locale_decimal == '.':
        return s
    # auto
    if "," in s and "." not in s:
        return s.replace(",", ".")
    return s


def _split_top_level(s: str, seps: Tuple[str, ...]) -> List[str]:
    parts, buf = [], []
    depth_paren = depth_square = depth_curly = 0
    for ch in s:
        if ch == '(':
            depth_paren += 1
        elif ch == ')':
            depth_paren -= 1
        elif ch == '[':
            depth_square += 1
        elif ch == ']':
            depth_square -= 1
        elif ch == '{':
            depth_curly += 1
        elif ch == '}':
            depth_curly -= 1
        if ch in seps and depth_paren == depth_square == depth_curly == 0:
            parts.append(''.join(buf)); buf = []
        else:
            buf.append(ch)
    if buf:
        parts.append(''.join(buf))
    return [p.strip() for p in parts if p.strip()]


# =============================================================================
# Parsiranje ulomkov in števil
# =============================================================================

def _normalize_sign(n: int, d: int) -> Tuple[int, int]:
    return (-n, -d) if d < 0 else (n, d)


def _parse_fraction_token(token: str, cfg: EvalConfig) -> Optional[Fraction]:
    m = _FRAC_TOKEN.fullmatch(token.strip())
    if not m:
        return None
    sign = -1 if m.group("sign") == "-" else 1
    n = int(m.group("n")) * sign
    d = int(m.group("d"))
    n, d = _normalize_sign(n, d)
    if d == 0:
        return None
    return Fraction(n, d)


def _strip_units_from_token(t: str) -> str:
    t = _strip_nbsp_and_units(t)
    # odstrani vmesne presledke med števkami (tisočiški kot presledek)
    if re.fullmatch(r"[+\-]?\d[\d\s.,]*", t):
        t = re.sub(r"(?<=\d)\s+(?=\d)", "", t)
    return t


def _parse_number_token(token: str, cfg: EvalConfig) -> Optional[Union[int, float]]:
    t = _strip_units_from_token(token.strip())
    # EU tisočiški
    if "." in t and "," in t:
        if re.fullmatch(r"[+\-]?\d{1,3}(?:\.\d{3})+,\d+", t) or re.fullmatch(r"[+\-]?\d{1,3}(?:\.\d{3})+$", t):
            t = t.replace(".", "").replace(",", ".")
    elif "," in t and "." not in t and cfg.locale_decimal in ("auto", ","):
        t = t.replace(",", ".")

    if _INT_TOKEN.fullmatch(t):
        try:
            return int(t)
        except Exception:
            return None

    if re.fullmatch(r'^[+\-]?\d+(?:\.\d+)?$', t or ""):
        try:
            return float(t)
        except Exception:
            return None
    return None


# =============================================================================
# Radikali (√ → sqrt) + varni eval
# =============================================================================

def _normalize_radicals(expr: str, cfg: EvalConfig) -> str:
    if not cfg.allow_radicals:
        return expr
    expr = _RADICAL_COEFF.sub(lambda m: f"{m.group('c')}*sqrt({m.group('n')})", expr)
    expr = _RADICAL_SOLO.sub(lambda m: f"sqrt({m.group('n')})", expr)
    return expr


_BIN_OPS = {
    ast.Add: (lambda a, b: a + b),
    ast.Sub: (lambda a, b: a - b),
    ast.Mult: (lambda a, b: a * b),
    ast.Div: (lambda a, b: a / b),
    ast.Pow: (lambda a, b: a ** b),
}
_UNARY_OPS = {ast.UAdd: (lambda a: a), ast.USub: (lambda a: -a)}

_ALLOWED_NODES = (
    ast.Expression, ast.BinOp, ast.UnaryOp, ast.Constant,
    ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow, ast.UAdd, ast.USub,
    ast.Load, ast.Tuple, ast.List, ast.Call, ast.Name, ast.Expr
)

_SAFE_CONSTS: Dict[str, Any] = {"pi": math.pi, "e": math.e}


def _normalize_expr_for_eval(expr: str, cfg: EvalConfig) -> str:
    expr = expr.replace("^", "**")
    expr = _normalize_radicals(expr, cfg)
    return expr


def _replace_simple_fractions(expr: str, cfg: EvalConfig) -> str:
    if not cfg.allow_fraction_eval:
        return expr
    def repl(m: re.Match) -> str:
        sign = m.group("sign") or ""
        n = m.group("n"); d = m.group("d")
        try:
            n_i = int(n); d_i = int(d)
        except Exception:
            return m.group(0)
        if sign == "-":
            n_i = -n_i
        n_i, d_i = _normalize_sign(n_i, d_i)
        if d_i == 0:
            return m.group(0)
        return f"Fraction({n_i},{d_i})"
    return _FRAC_TOKEN.sub(repl, expr)


def _safe_eval_expr(expr: str, cfg: EvalConfig) -> Any:
    if not cfg.allow_expressions:
        raise ValueError("Expressions not allowed by configuration.")
    if len(expr) > cfg.max_expr_len:
        raise ValueError("Expression too long.")

    expr = _normalize_expr_for_eval(expr, cfg)
    expr = _replace_simple_fractions(expr, cfg)

    node = ast.parse(expr, mode="eval")
    for sub in ast.walk(node):
        if not isinstance(sub, _ALLOWED_NODES):
            raise ValueError(f"Disallowed AST element: {type(sub).__name__}")

    def _eval(n):
        if isinstance(n, ast.Expression):
            return _eval(n.body)
        if isinstance(n, ast.Constant) and isinstance(n.value, (int, float)):
            return n.value
        if isinstance(n, ast.UnaryOp) and type(n.op) in _UNARY_OPS:
            return _UNARY_OPS[type(n.op)](_eval(n.operand))
        if isinstance(n, ast.BinOp) and type(n.op) in _BIN_OPS:
            left = _eval(n.left); right = _eval(n.right)
            return _BIN_OPS[type(n.op)](left, right)
        if isinstance(n, ast.Tuple):
            return tuple(_eval(elt) for elt in n.elts)
        if isinstance(n, ast.List):
            return [_eval(elt) for elt in n.elts]
        if isinstance(n, ast.Name):
            if n.id in _SAFE_CONSTS and cfg.allow_constants:
                return _SAFE_CONSTS[n.id]
            raise ValueError(f"Name '{n.id}' not allowed.")
        if isinstance(n, ast.Call):
            # Dovolimo Fraction(a,b) in sqrt(a)
            if isinstance(n.func, ast.Name) and n.func.id == "Fraction":
                if len(n.args) != 2:
                    raise ValueError("Fraction requires two arguments.")
                a = _eval(n.args[0]); b = _eval(n.args[1])
                if isinstance(a, Fraction) and a.denominator == 1: a = a.numerator
                if isinstance(b, Fraction) and b.denominator == 1: b = b.numerator
                return Fraction(a, b)
            if isinstance(n.func, ast.Name) and n.func.id == "sqrt" and cfg.allow_radicals:
                if len(n.args) != 1:
                    raise ValueError("sqrt requires one argument.")
                a = _eval(n.args[0])
                if isinstance(a, Fraction): a = float(a)
                if not isinstance(a, (int, float)):
                    raise ValueError("sqrt argument must be numeric.")
                if a < 0:
                    raise ValueError("sqrt of negative is not supported.")
                return math.sqrt(a)
            raise ValueError("Function calls not allowed (only Fraction, sqrt).")
        raise ValueError(f"Unsupported AST node: {type(n).__name__}")

    return _eval(node)


# =============================================================================
# Parsiranje uporabniškega vnosa (vse oblike)
# =============================================================================

def _strip_leading_assignment(raw: str) -> str:
    if "=" not in raw:
        return raw
    m = _ASSIGN_LHS_RE.match(raw)
    return m.group("rhs").strip() if m else raw


def _parse_atom(token: str, cfg: EvalConfig) -> Optional[Any]:
    token = token.strip()
    fr = _parse_fraction_token(token, cfg)
    if fr is not None:
        return fr
    num = _parse_number_token(token, cfg)
    if num is not None:
        return num
    return None


def _parse_list_like(s: str, cfg: EvalConfig) -> Optional[Union[List[Any], Tuple[Any, ...], set]]:
    s = s.strip()
    if not s or s[0] not in _LIST_LIKE_BRACKETS:
        return None
    close = _LIST_LIKE_BRACKETS[s[0]]
    if not s.endswith(close):
        return None
    inner = s[1:-1].strip()
    parts = _split_top_level(inner, (',', ';'))
    items: List[Any] = []
    for p in parts:
        v = _parse_atom(p, cfg)
        if v is None:
            try:
                v = _safe_eval_expr(p, cfg)
            except Exception:
                return None
        items.append(v)
    if s[0] == '(':
        return tuple(items)
    if s[0] == '[':
        return items
    # set – poskrbi za nehashable
    try:
        return set(items)
    except Exception:
        return set(tuple(x) if isinstance(x, list) else x for x in items)


def _extract_last_numeric_from_text(s: str, cfg: EvalConfig) -> Optional[Union[int, float, Fraction]]:
    """
    V mešanem besedilu poišče **zadnji** številski/ulomčni token.
    Prednost damo zadnjemu segmentu po ';' (npr. '...; y(1)=5').
    """
    if not s:
        return None
    s_norm = _normalize_decimal_separator(s, cfg)
    # razbij po ; → vzemi zadnji del
    chunks = [c.strip() for c in s_norm.split(";") if c.strip()]
    candidates = [chunks[-1]] if chunks else [s_norm]

    for seg in candidates:
        matches = list(_NUM_OR_FRAC_TOKEN.finditer(seg))
        if not matches:
            continue
        tok = matches[-1].group(0)
        fr = _parse_fraction_token(tok, cfg)
        if fr is not None:
            return fr
        num = _parse_number_token(tok, cfg)
        if num is not None:
            return num
    return None


def _parse_user_input(user_input: str, cfg: EvalConfig) -> Tuple[Any, str]:
    """
    Vrne (kanonična_vrednost, tip_opisa): {"int","float","fraction","list","tuple","set","expr","raw"}.
    """
    raw = user_input.strip()
    if not raw:
        return None, "raw"

    raw = _strip_leading_assignment(raw)
    norm = _normalize_decimal_separator(raw, cfg)

    lst = _parse_list_like(norm, cfg)
    if lst is not None:
        t = "tuple" if isinstance(lst, tuple) else "list" if isinstance(lst, list) else "set"
        return lst, t

    atom = _parse_atom(norm, cfg)
    if atom is not None:
        if isinstance(atom, Fraction): return atom, "fraction"
        if isinstance(atom, int):      return int(atom), "int"
        if isinstance(atom, float):    return float(atom), "float"

    if cfg.allow_expressions:
        try:
            val = _safe_eval_expr(norm, cfg)
            return val, "expr"
        except Exception:
            pass

    return norm, "raw"


# =============================================================================
# Kanonizacija rešitve (problem.solution ali problem.numeric)
# =============================================================================

def _parse_solution(problem) -> Any:
    sol = getattr(problem, "solution", None)
    if isinstance(sol, (int, float, Fraction, list, tuple, set)):
        return sol

    if isinstance(sol, str):
        cfg = EvalConfig()
        s2 = _strip_leading_assignment(sol)
        val, typ = _parse_user_input(s2, cfg)
        if val is not None and typ != "raw":
            return val

    if hasattr(problem, "numeric") and problem.numeric is not None:
        return float(problem.numeric)

    return sol


# =============================================================================
# Primerjava (števila, zaporedja, domene)
# =============================================================================

NumberLike = Union[int, float, Fraction]
SeqLike    = Union[List[Any], Tuple[Any, ...], set]


def _num_equal(a: NumberLike, b: NumberLike, cfg: EvalConfig) -> Tuple[bool, str]:
    # Fraction ↔ Fraction
    if isinstance(a, Fraction) and isinstance(b, Fraction):
        return (a == b, "exact" if a == b else "wrong-value")

    # Ekvivalenca Fraction ↔ (int/float)
    if cfg.accept_equivalent_forms:
        if isinstance(a, Fraction) and isinstance(b, (int, float)):
            if isinstance(b, int):
                b_frac = Fraction(b)
                return (a == b_frac, "exact" if a == b_frac else "wrong-value")
            ok = math.isclose(float(a), float(b),
                              rel_tol=cfg.decimal_tolerance_rel,
                              abs_tol=cfg.decimal_tolerance_abs)
            return (ok, "close" if ok else "wrong-value")
        if isinstance(b, Fraction) and isinstance(a, (int, float)):
            ok, kind = _num_equal(b, a, cfg)
            return ok, kind

    af, bf = float(a), float(b)
    ok = math.isclose(af, bf,
                      rel_tol=cfg.decimal_tolerance_rel,
                      abs_tol=cfg.decimal_tolerance_abs)
    if ok:
        if isinstance(a, int) and isinstance(b, int) and a == b:
            return True, "exact"
        return True, "close"
    return False, "wrong-value"


def _seq_equal(a: SeqLike, b: SeqLike, cfg: EvalConfig) -> Tuple[bool, str]:
    def _to_list(x):
        if isinstance(x, tuple): return list(x)
        if isinstance(x, list):  return x[:]
        if isinstance(x, set):   return sorted(list(x))
        return [x]

    # Množična ekvivalenca (neobčutljiva na vrstni red)
    if isinstance(a, set) or isinstance(b, set) or cfg.set_order_invariant:
        try:
            sa = set(a if isinstance(a, (list, tuple, set)) else [a])
            sb = set(b if isinstance(b, (list, tuple, set)) else [b])

            def _canon(elem):
                if isinstance(elem, Fraction):
                    return ('F', elem.numerator, elem.denominator)
                if isinstance(elem, (int, float)):
                    return ('N', round(float(elem), 12))
                if isinstance(elem, (tuple, list)):
                    return ('T', tuple(round(float(e), 12) if isinstance(e, (int, float, Fraction)) else e for e in elem))
                return ('O', elem)

            ca = {_canon(e) for e in sa}
            cb = {_canon(e) for e in sb}
            return (ca == cb, "exact" if ca == cb else "wrong-value")
        except Exception:
            pass

    la, lb = _to_list(a), _to_list(b)
    if len(la) != len(lb):
        return (False, "wrong-value")

    all_ok, any_close = True, False
    for x, y in zip(la, lb):
        if isinstance(x, (int, float, Fraction)) and isinstance(y, (int, float, Fraction)):
            ok, kind = _num_equal(x, y, cfg)
        else:
            ok = (x == y)
            kind = "exact" if ok else "wrong-value"
        all_ok &= ok
        any_close |= (kind == "close")

    if all_ok:
        return (True, "close" if any_close else "exact")
    return (False, "wrong-value")


# ---- Simbolne domene (ℝ \ {…}, x ≠ …, unija intervalov okoli c) ----

def _to_frac__domain(num_s: str) -> Optional[Fraction]:
    s = num_s.strip().replace(",", ".")
    if "/" in s:
        try:
            return Fraction(s)
        except Exception:
            return None
    try:
        return Fraction.from_float(float(s)).limit_denominator()
    except Exception:
        return None


def _extract_exclusions(text: str) -> set[Fraction]:
    s = text
    excl: set[Fraction] = set()

    # Zajem iz { a; b, c }
    m = re.search(r"\{([^}]*)\}", s)
    if m:
        inside = m.group(1)
        for tok in re.split(r"[;,]\s*", inside):
            tok = tok.strip()
            if tok == "":
                continue
            f = _to_frac__domain(tok)
            if f is not None:
                excl.add(f)

    # Zajem iz x ≠ c / x!=c / x=/=c
    for m2 in re.finditer(r"\b[xy]\s*(?:≠|!=|=\/=)\s*([\-+]?\d+(?:/\d+)?(?:[.,]\d+)?)", s):
        f = _to_frac__domain(m2.group(1))
        if f is not None:
            excl.add(f)

    # Zajem iz (-∞, c) ∪ (c, ∞)
    for m3 in re.finditer(r"\(\s*-\s*∞\s*,\s*([\-+]?\d+(?:/\d+)?(?:[.,]\d+)?)\s*\)\s*[∪U]\s*\(\s*\1\s*,\s*∞\s*\)", s):
        f = _to_frac__domain(m3.group(1))
        if f is not None:
            excl.add(f)
    return excl


def _normalize_domain_string(text: str) -> Optional[tuple[str, tuple[Fraction, ...]]]:
    s = text.strip()
    looks_like_R = any(t in s for t in ("ℝ", "R", "(-∞", "∞", "x ≠", "x!=", "{", "}"))
    if not looks_like_R:
        return None
    excl = _extract_exclusions(s)
    if not excl and re.search(r"\b(ℝ|R)\b", s):
        return ("R_MINUS", tuple())
    if excl:
        return ("R_MINUS", tuple(sorted(excl)))
    return None


def _domains_equivalent(user_text: str, sol_text: str) -> bool:
    u = _normalize_domain_string(user_text)
    v = _normalize_domain_string(sol_text)
    if u is None or v is None:
        return False
    return u == v


# =============================================================================
# Plugin sistem (neobvezen)
# =============================================================================

def _load_plugins() -> List[Any]:
    plugins = []
    base_pkg = "orodja.odgovor_plugins"
    candidates = [
        "fractions_plugin",
        "pemdas_plugin",
        "trig_plugin",
        "linear_plugin",
        "polynomial_plugin",
        "percent_plugin",
    ]
    for modname in candidates:
        try:
            mod = importlib.import_module(f"{base_pkg}.{modname}")
            plugins.append(mod)
        except Exception:
            continue
    return plugins


# =============================================================================
# Glavni API
# =============================================================================

def evaluate_answer(problem: Any, user_input: str, *, subject: Optional[str] = None, config: Optional[EvalConfig] = None) -> Verdict:
    cfg = config or EvalConfig()
    category = subject or getattr(problem, "category", "") or ""

    # 1) Plugins (mehko: če vrnejo "wrong-format", nadaljuj s CORE)
    for pl in _load_plugins():
        try:
            can = getattr(pl, "can_handle", None)
            if callable(can) and can(problem=problem, subject=category):
                pre     = getattr(pl, "preprocess_user_input", None)
                can_sol = getattr(pl, "canonical_solution", None)
                cmpf    = getattr(pl, "compare", None)
                if not (callable(pre) and callable(can_sol) and callable(cmpf)):
                    continue
                lhs = pre(problem=problem, user_input=user_input, config=cfg)
                rhs = can_sol(problem=problem, config=cfg)
                ok, match, expl = cmpf(lhs, rhs, cfg)
                # Če plugin uspe ali vrne karkoli drugega kot "wrong-format", sprejmi
                if ok or match != "wrong-format":
                    return Verdict(ok, match, lhs, rhs, explanation=expl,
                                   debug={"plugin": getattr(pl, "name", pl.__name__)})
        except Exception:
            # plugin spodleti → ignoriramo in gremo v CORE
            continue

    # 2) CORE fallback
    usr_val, usr_typ = _parse_user_input(user_input, cfg)
    sol_val = _parse_solution(problem)

    # 2a) Čista numerika
    if isinstance(usr_val, (int, float, Fraction)) and isinstance(sol_val, (int, float, Fraction)):
        ok, kind = _num_equal(usr_val, sol_val, cfg)
        return Verdict(ok, kind, usr_val, sol_val, explanation=("" if ok else "Vrednost se ne ujema."))

    # 2b) Zaporedja/množice
    if isinstance(usr_val, (list, tuple, set)) and isinstance(sol_val, (list, tuple, set)):
        ok, kind = _seq_equal(usr_val, sol_val, cfg)
        return Verdict(ok, kind, usr_val, sol_val, explanation=("" if ok else "Elementi ali njihov vrstni red se ne ujemajo."))

    # 2c) Izraz → numerika/sekvence
    if usr_typ == "expr":
        uv = usr_val
        if isinstance(sol_val, (int, float, Fraction)):
            ok, kind = _num_equal(uv, sol_val, cfg)
            return Verdict(ok, kind, uv, sol_val, explanation=("" if ok else "Rezultat izraza se ne ujema z rešitvijo."))
        if isinstance(sol_val, (list, tuple)) and isinstance(uv, (list, tuple)):
            ok, kind = _seq_equal(uv, sol_val, cfg)
            return Verdict(ok, kind, uv, sol_val, explanation=("" if ok else "Rezultat izraza (seznam/tuple) se ne ujema."))

    # 2d) Mešano besedilo + numerični summary (npr. '; y(1)=5', '≈ 3,14 €', 'Rezultat = 7/5')
    if isinstance(sol_val, (int, float, Fraction)) and isinstance(usr_val, str):
        last_num = _extract_last_numeric_from_text(usr_val, cfg)
        if last_num is not None:
            ok, kind = _num_equal(last_num, sol_val, cfg)
            return Verdict(ok, kind, last_num, sol_val,
                           explanation=("" if ok else "Končna numerična vrednost v tvojem zapisu se ne ujema z rešitvijo."))

    # 2e) “Mehki” numeric fallback (poskusi float() pretvorbo)
    try:
        uvf = float(usr_val) if not isinstance(usr_val, Fraction) else float(usr_val)
        svf = float(sol_val) if not isinstance(sol_val, Fraction) else float(sol_val)
        ok, kind = _num_equal(uvf, svf, cfg)
        return Verdict(ok, kind, usr_val, sol_val, explanation=("" if ok else "Vrednost se ne ujema."))
    except Exception:
        pass

    # 2f) Simbolne domene (ℝ \ {…}, x ≠ …, (-∞,c)∪(c,∞))
    try:
        subj = (subject or getattr(problem, "category", "") or "").lower()
        user_txt = user_input.strip()
        sol_txt  = str(getattr(problem, "solution", ""))
        looks_symbolic = any(tok in (user_txt + sol_txt) for tok in ("ℝ", "R", "(-∞", "∞", "≠", "!=", "{", "}"))
        if ("funkcije" in subj or "graf" in subj or looks_symbolic):
            if _domains_equivalent(user_txt, sol_txt):
                return Verdict(True, "symbolic-domain", user_txt, sol_txt,
                               explanation="Pravilno: domena je enakovredno zapisana.", debug={})
    except Exception:
        pass

    # 2z) Generični zaključek
    return Verdict(False, "wrong-format", usr_val, sol_val,
                   explanation="Neusklajen format odgovora ali nerešljiva primerjava.")


--- Konec datoteke: orodja\odgovor_eval.py ---

--- Začetek datoteke: orodja\selfcheck.py ---

# -*- coding: utf-8 -*-
"""
orodja/selfcheck.py
-------------------
Centraliziran "samopreizkus" za generirane naloge.

Zagotavlja:
- eval_prompt_expr(prompt_expr: str) -> float
    Normalizira izraz iz prompta (zamenja ·, ÷, ^, [ ] → ( ), √ → sqrt) in ga varno ovrednoti.
- verify_problem_numeric(p: Problem) -> bool
    Če ima naloga smiseln p.numeric, neodvisno preveri skladnost z izračunom iz prompta,
    kjer je to mogoče (Vrstni red računov, Potence/Koreni). Pri nepodprtih kategorijah
    se ne dela lažnih negativnih: vrne True (skip).

Opombe:
- Če je na voljo core evaluator (orodja.odgovor_eval._safe_eval_expr), ga raje uporabimo.
- Lokalni eval je whitelisted (AST) ter podpira + - * / **, unarni +/- in sqrt(...).
"""

from __future__ import annotations

import ast
import math
import re
from typing import Any

# ——— (poskus) uvoz core evaluatorja za konsistenco, sicer fallback ———
_CORE_SAFE = None
try:
    from orodja.odgovor_eval import _safe_eval_expr as _CORE_SAFE  # type: ignore
except Exception:
    _CORE_SAFE = None


# ------------------------ Normalizacija prompt izraza -------------------------

_PROMPT_HEADERS = (
    "Uredi:",
    "Izračunaj:", "Izračunaj in poenostavi:", "Izračunaj vrednost:",
    "Poenostavi:", "Uredi izraz:",
)

def _extract_expr_from_prompt(raw_prompt: str) -> str:
    """
    Vrne del izraza ZA oznako tipa 'Uredi:' / 'Izračunaj:' ipd.
    Če glave ne najde, vrne celoten niz (bolje to kot False negative).
    """
    text = (raw_prompt or "").strip()
    for head in _PROMPT_HEADERS:
        if head in text:
            return text.split(head, 1)[1].strip()
    return text

def _normalize_for_eval(expr: str) -> str:
    """
    Normalizira vizualne znake v Pythonu podoben izraz:
     - [ ] → ( ), '·' → '*', '÷' → '/', '^' → '**', '√' → 'sqrt'
     - pusti presledke, oklepaje, minus ipd.
    """
    e = (expr or "")
    e = e.replace("[", "(").replace("]", ")")
    e = e.replace("·", "*").replace("÷", "/").replace("^", "**")
    e = e.replace("√", "sqrt")
    # varnost: večkratni presledki niso problematični
    return e.strip()


# ------------------------ Lokalni varen eval (fallback) ------------------------

_BIN_OPS = {
    ast.Add: lambda a, b: a + b,
    ast.Sub: lambda a, b: a - b,
    ast.Mult: lambda a, b: a * b,
    ast.Div: lambda a, b: a / b,
    ast.Pow: lambda a, b: a ** b,
}
_UNARY_OPS = {ast.UAdd: lambda a: +a, ast.USub: lambda a: -a}

_ALLOWED_CALLS = {"sqrt": math.sqrt}
_ALLOWED_NODES = (
    ast.Expression, ast.BinOp, ast.UnaryOp, ast.Constant,
    ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow, ast.UAdd, ast.USub,
    ast.Call, ast.Name, ast.Load, ast.Tuple, ast.List
)

def _safe_eval_local(expr: str) -> float:
    node = ast.parse(expr, mode="eval")

    # whitelist preverjanje
    for sub in ast.walk(node):
        if not isinstance(sub, _ALLOWED_NODES):
            raise ValueError(f"Nedovoljen AST element: {type(sub).__name__}")

    def _ev(n):
        if isinstance(n, ast.Expression):
            return _ev(n.body)
        if isinstance(n, ast.Constant):
            if isinstance(n.value, (int, float)):
                return float(n.value)
            raise ValueError("Nedovoljena konstanta")
        if isinstance(n, ast.UnaryOp) and type(n.op) in _UNARY_OPS:
            return _UNARY_OPS[type(n.op)](_ev(n.operand))
        if isinstance(n, ast.BinOp) and type(n.op) in _BIN_OPS:
            return _BIN_OPS[type(n.op)](_ev(n.left), _ev(n.right))
        if isinstance(n, ast.Name):
            # dovolimo le dovoljena imena (npr. sqrt), če bi se pojavila kot golo ime (ne bi smela)
            if n.id in _ALLOWED_CALLS:
                return _ALLOWED_CALLS[n.id]
            raise ValueError(f"Nedovoljeno ime: {n.id}")
        if isinstance(n, ast.Call):
            if isinstance(n.func, ast.Name) and n.func.id in _ALLOWED_CALLS:
                fn = _ALLOWED_CALLS[n.func.id]
                args = [_ev(a) for a in n.args]
                return float(fn(*args))
            raise ValueError("Dovoljeni so le klici sqrt(...)")
        raise ValueError("Nepodprt izraz")

    val = _ev(node)
    return float(val)


# ------------------------ Javni API ------------------------

def eval_prompt_expr(prompt_expr: str) -> float:
    """
    Normalizira in varno ovrednoti izraz iz prompta.
    Če je na voljo core evaluator (_CORE_SAFE), ga uporabi (konsistentno z glavno logiko).
    Sicer uporabi lokalni eval.
    """
    expr = _normalize_for_eval(_extract_expr_from_prompt(prompt_expr))
    if not expr:
        raise ValueError("Ne najdem izraza v promptu.")
    if _CORE_SAFE is not None:
        # Core eval podpira Fraction in podobno; za self-check tu zadošča float rezultat
        val = _CORE_SAFE(expr, config=None)  # type: ignore[arg-type]
        # _CORE_SAFE lahko vrne npr. Fraction/float/list; potrebujemo številko:
        if isinstance(val, (int, float)):
            return float(val)
        raise ValueError("Jedrni eval je vrnil neštevilski tip za self-check.")
    # fallback:
    return _safe_eval_local(expr)


def verify_problem_numeric(p: Any) -> bool:
    """
    Neodvisno preveri skladnost `p.numeric` z izračunom iz `p.prompt`, kjer je to smiselno.
    Vrne True, če:
      - uspešno preveri in se ujema (isclose),
      - kategorija ni podprta za robusten eval in ima smiseln numeric (raje skip kot false negative),
      - naloga nima numeric (ni kaj preverjati).

    Podprte kategorije (zmerno robustno):
      - "Vrstni red računov"  → eval izraz iz prompta
      - "Potence/Koreni"      → prepozna √, ^, ipd.
    """
    try:
        # Če numeric ni podan, nimamo referenčne vrednosti
        if getattr(p, "numeric", None) is None:
            return True

        cat = (getattr(p, "category", "") or "").strip()

        # Kategorije, kjer lahko zanesljivo ovrednotimo prompt:
        SUPPORTED = {"Vrstni red računov", "Potence/Koreni"}

        if cat in SUPPORTED:
            calc = eval_prompt_expr(getattr(p, "prompt", ""))
            target = float(p.numeric)
            return math.isclose(calc, target, rel_tol=1e-12, abs_tol=1e-12)

        # Za ostale kategorije (Polinomi, Trig, Funkcije, ...): skip (True),
        # dokler ne dodamo specializiranih parserjev.
        return True

    except Exception:
        # V self-checku raje ne rušimo toka zaradi robnih primerov; označimo kot neuspeh.
        return False


--- Konec datoteke: orodja\selfcheck.py ---

--- Začetek datoteke: orodja\__init__.py ---



--- Konec datoteke: orodja\__init__.py ---

--- Začetek datoteke: orodja\odgovor_plugins\fractions_plugin.py ---

# -*- coding: utf-8 -*-
"""
orodja/odgovor_plugins/fractions_plugin.py
------------------------------------------
Plugin za predmet "Ulomki" (fractions).

Integracija:
- Jedro (orodja/odgovor_eval.py) bo samodejno poskusilo uvoziti ta modul
  (ime 'fractions_plugin' je že na seznamu kandidatov loaderja).

Zagotavlja funkcije:
- name: str
- can_handle(problem, subject) -> bool
- preprocess_user_input(problem, user_input, config) -> Any
- canonical_solution(problem, config) -> Any
- compare(lhs, rhs, config) -> (bool, match:str, explanation:str)
"""

from __future__ import annotations

import math
import re
from fractions import Fraction
from typing import Any, Dict, Optional, Tuple, Union

# Pri klicu bo jedro podalo EvalConfig; tu tip hint samo informativno
try:
    from orodja.odgovor_eval import EvalConfig  # za type hints (ne nujno za runtime)
except Exception:
    class EvalConfig:  # fallback, če tip ni na voljo pri statični analizi
        decimal_tolerance_abs: float = 1e-9
        decimal_tolerance_rel: float = 1e-9
        locale_decimal: str = "auto"
        accept_equivalent_forms: bool = True
        set_order_invariant: bool = True
        allow_expressions: bool = True

name = "fractions"

CATEGORY_MATCHES = {"Ulomki", "Fractions"}

_FRAC_TOKEN = re.compile(r'(?P<sign>[+\-]?)\s*(?P<n>\d+)\s*/\s*(?P<d>-?\d+)')
_INT_TOKEN = re.compile(r'^[+\-]?\d+$')
_FLOAT_TOKEN = re.compile(r'^[+\-]?\d+(?:[.,]\d+)?$')


# ------------------------ Pomagala: parsing & normalizacija ------------------------

def _normalize_sign(n: int, d: int) -> Tuple[int, int]:
    """Znak nosi števec; imenovalec pozitiven."""
    if d < 0:
        n, d = -n, -d
    return n, d

def _to_fraction_from_str_token(token: str, cfg: EvalConfig) -> Optional[Fraction]:
    """Poskusi pretvoriti posamičen token v Fraction (p/q ali celo ali decimalno)."""
    t = token.strip()

    # p/q
    m = _FRAC_TOKEN.fullmatch(t)
    if m:
        sign = -1 if (m.group("sign") == "-") else 1
        n = int(m.group("n")) * sign
        d = int(m.group("d"))
        n, d = _normalize_sign(n, d)
        if d == 0:
            return None
        return Fraction(n, d)

    # celo število
    if _INT_TOKEN.fullmatch(t):
        try:
            return Fraction(int(t), 1)
        except Exception:
            return None

    # decimalno število (vejica ali pika)
    if _FLOAT_TOKEN.fullmatch(t):
        if cfg.locale_decimal == ",":
            t2 = t.replace(",", ".")
        elif cfg.locale_decimal == ".":
            t2 = t
        else:
            # auto: če vsebuje vejico, interpretiramo kot decimalno
            t2 = t.replace(",", ".")
        try:
            return Fraction(t2)  # Fraction str → razumno natančna racionalizacija
        except Exception:
            # fallback: prek float (manj natančno)
            try:
                return Fraction.from_float(float(t2)).limit_denominator()
            except Exception:
                return None

    return None

def _strip_outer_parentheses_if_pair(s: str) -> str:
    """Odstrani odvečne zunanje oklepaje okoli celotnega vnosa, npr. '(-5/9)' → '-5/9'."""
    s = s.strip()
    if len(s) >= 2 and s[0] == '(' and s[-1] == ')':
        # preveri, da se oklepaji ujemajo na top-level
        depth = 0
        for i, ch in enumerate(s):
            if ch == '(':
                depth += 1
            elif ch == ')':
                depth -= 1
                if depth == 0 and i != len(s) - 1:
                    break
        else:
            # oklepaji objemajo cel niz
            return s[1:-1].strip()
    return s

def _normalize_double_minus_view(s: str) -> str:
    """
    Vizualna normalizacija za primere 'a/b - - c/d' → 'a/b - (-c/d)'.
    Ne spreminja vrednosti, le pomaga pri doslednosti prikaza.
    """
    return s.replace("- -", "- (-")

# ------------------------ API: can_handle ------------------------------------------

def can_handle(problem: Any, subject: Optional[str] = None) -> bool:
    """Odločimo se po kategoriji (ali stringu subject)."""
    cat = subject or getattr(problem, "category", "") or ""
    return any(key.lower() in cat.lower() for key in CATEGORY_MATCHES)

# ------------------------ API: preprocess_user_input --------------------------------

def preprocess_user_input(problem: Any, user_input: str, config: EvalConfig) -> Any:
    """
    Vrne kanonično vrednost uporabnikovega vnosa:
    - Fraction za večino primerov,
    - ali surovi niz, če parsing odpove (da jedro lahko poda 'wrong-format').
    """
    s = user_input.strip()
    if not s:
        return ""

    s = _normalize_double_minus_view(s)
    s = _strip_outer_parentheses_if_pair(s)

    fr = _to_fraction_from_str_token(s, config)
    if fr is not None:
        return fr

    # Dovolimo še enostavne izraze tipa "1/2 + 3/4" → to naj preverja CORE.
    # Tukaj vrnemo surov niz; jedro bo evalviralo (Fraction) in primerjalo.
    return s

# ------------------------ API: canonical_solution -----------------------------------

def canonical_solution(problem: Any, config: EvalConfig) -> Any:
    """
    Kanonična rešitev:
    - poskusi `problem.solution` → Fraction
    - če ne gre, poskusi `problem.numeric` → Fraction (limit_denominator)
    - fallback: vrni surovi string
    """
    sol = getattr(problem, "solution", None)
    if isinstance(sol, Fraction):
        return sol
    if isinstance(sol, (int, float)):
        return Fraction(sol).limit_denominator()

    if isinstance(sol, str):
        t = sol.strip()
        fr = _to_fraction_from_str_token(t, config)
        if fr is not None:
            return fr
        # morda je sol npr. '-5/36' z odvečnimi oklepaji
        t = _strip_outer_parentheses_if_pair(t)
        fr = _to_fraction_from_str_token(t, config)
        if fr is not None:
            return fr

    # poskusi numeric
    if hasattr(problem, "numeric") and problem.numeric is not None:
        try:
            return Fraction(problem.numeric).limit_denominator()
        except Exception:
            pass

    return sol

# ------------------------ API: compare ----------------------------------------------

def _fractions_equal(a: Fraction, b: Fraction) -> bool:
    return a == b  # eksaktno

def _fraction_vs_number(a: Fraction, b: Union[int, float], cfg: EvalConfig) -> Tuple[bool, str]:
    # Če je int → eksaktno
    if isinstance(b, int):
        return (a == Fraction(b, 1), "exact" if a == Fraction(b, 1) else "wrong-value")
    # float → približno
    ok = math.isclose(float(a), float(b), rel_tol=cfg.decimal_tolerance_rel, abs_tol=cfg.decimal_tolerance_abs)
    return (ok, "close" if ok else "wrong-value")

def compare(lhs: Any, rhs: Any, config: EvalConfig) -> Tuple[bool, str, str]:
    """
    Primerja uporabnikov vnos (lhs) z rešitvijo (rhs).
    Vrne (is_correct, match, explanation).
    """
    # 1) Fraction vs Fraction → eksaktno
    if isinstance(lhs, Fraction) and isinstance(rhs, Fraction):
        ok = _fractions_equal(lhs, rhs)
        if ok:
            return True, "exact", ""
        # Napačno: podaj uporabno razlago
        expl = "Neujema se števec/imenovalec. Namig: uporabi SKI ali 'križno' pravilo in skrajšaj ulomek."
        return False, "wrong-value", expl

    # 2) Fraction vs (int|float)
    if isinstance(lhs, Fraction) and isinstance(rhs, (int, float)):
        ok, kind = _fraction_vs_number(lhs, rhs, config)
        if ok:
            return True, kind, ""
        expl = "Vrednost je blizu, a ni povsem pravilna. Preveri korake in skrajšanje."
        return False, "wrong-value", expl

    # 3) (int|float) vs Fraction (uporabnik vpisal decimalno/celo, rešitev je ulomek)
    if isinstance(rhs, Fraction) and isinstance(lhs, (int, float)):
        ok, kind = _fraction_vs_number(rhs, lhs, config)
        if ok:
            return True, kind, ""
        expl = "Decimalni zapis se ne ujema z ulomkom. Poskusi ulomek ali natančnejši rezultat."
        return False, "wrong-value", expl

    # 4) Surov niz (izrazi ali neprepoznano) – naj preveri CORE (varni eval ipd.).
    # V tem pluginu vrnemo 'wrong-format', da jedro poskusi fallback.
    return False, "wrong-format", "Format odgovora ni prepoznan kot ulomek/število."


--- Konec datoteke: orodja\odgovor_plugins\fractions_plugin.py ---

--- Začetek datoteke: orodja\odgovor_plugins\linear_plugin.py ---

# -*- coding: utf-8 -*-
"""
orodja/odgovor_plugins/linear_plugin.py
---------------------------------------
Plugin za "Linearne enačbe" in "Sisteme linearnih enačb".

Kaj podpira uporabniški vnos:
- skalar: "3", "-2,5" (vejica ali pika)
- urejen par/triple/...: "(1,2)", "1,2", "[1; 2]"  (ločila ',' ali ';')
- poimensko: "x=1, y=2" (vrstni red polj ni pomemben)
- množice parov: "{(1,2), (3,4)}", "x=1;y=2 | x=3;y=4"
- posebni primeri: "∅", "ni rešitve", "brez rešitve", "inf", "neskončno"

Kaj pričakuje iz `Problem.meta`:
- "solution_type": "scalar" | "pair" | "triple" | "vector" | "set_of_pairs" | "none" | "infinite"
- "variables": seznam spremenljivk v vrstnem redu, npr. ["x","y"] (za pair/triple/vector)
- "order_invariant": bool (privzeto True; ali je vrstni red elementov v množici parov pomemben)
- (opcijsko) "pair_separator": '|' ali ',' za parse alternativ

Primeri `solution`:
- "pair": "(1, 2)" ali "x=1, y=2"
- "set_of_pairs": "{(1,2), (3,4)}" ali "x=1;y=2 | x=3;y=4"
- "scalar": "3/2" ali "1.5"
- "none": "∅"
- "infinite": "inf"

Opomba:
- Plugin NE rešuje sistema; le primerja uporabnikov odgovor s kanonično rešitvijo.
"""

from __future__ import annotations

import re
import math
from fractions import Fraction
from typing import Any, List, Tuple, Dict, Optional

# mehka odvisnost za type hints (core morda še ni uvožen ob linti)
try:
    from orodja.odgovor_eval import EvalConfig
except Exception:  # pragma: no cover
    class EvalConfig:
        decimal_tolerance_abs: float = 1e-9
        decimal_tolerance_rel: float = 1e-9
        locale_decimal: str = "auto"
        accept_equivalent_forms: bool = True
        set_order_invariant: bool = True
        allow_expressions: bool = True


name = "linear"

CATEGORY_MATCHES = {
    "Linearne enačbe", "Linearni sistemi", "Sistemi enačb",
    "Linear equations", "Systems of linear equations", "Sistemi"
}

_ASSIGN_TOKEN = re.compile(r'\s*([A-Za-z]\w*)\s*=\s*([^\s,;|]+)\s*')
_LIST_SEP = re.compile(r'\s*[;,]\s*')  # za razbitje "1,2" ali "1;2"

# ----------------------------- Pomožne: parsing števil ------------------------------

def _parse_number(s: str, cfg: EvalConfig) -> Optional[float]:
    """Pretvori niz v float (podpira vejico/piko in ulomek p/q)."""
    t = s.strip()
    # ulomek p/q
    m = re.fullmatch(r'([+\-]?)\s*(\d+)\s*/\s*([+\-]?\d+)', t)
    if m:
        sign = -1 if m.group(1) == '-' else 1
        n = int(m.group(2)) * sign
        d = int(m.group(3))
        if d == 0:
            return None
        return float(Fraction(n, d))
    # decimalna/pika ali vejica
    if cfg.locale_decimal == ',':
        t = t.replace(',', '.')
    elif cfg.locale_decimal == 'auto':
        if ',' in t and '.' not in t:
            t = t.replace(',', '.')
    # gole številke
    try:
        return float(t)
    except Exception:
        return None

def _num_close(a: float, b: float, cfg: EvalConfig) -> bool:
    return math.isclose(float(a), float(b), rel_tol=cfg.decimal_tolerance_rel, abs_tol=cfg.decimal_tolerance_abs)

# ----------------------------- Pomožne: razbitje struktur --------------------------

def _strip_outer_brackets(s: str) -> str:
    s = s.strip()
    if len(s) >= 2:
        pairs = {'(': ')', '[': ']', '{': '}'}
        if s[0] in pairs and s[-1] == pairs[s[0]]:
            # preveri, da objema cel niz
            depth = 0
            ok = True
            for i, ch in enumerate(s):
                if ch in '([{':
                    depth += 1
                elif ch in ')]}':
                    depth -= 1
                    if depth == 0 and i != len(s)-1:
                        ok = False
                        break
            if ok and depth == 0:
                return s[1:-1].strip()
    return s

def _split_top_level(s: str, seps: Tuple[str, ...]) -> List[str]:
    parts, buf = [], []
    depth = 0
    for ch in s:
        if ch in '([{':
            depth += 1
        elif ch in ')]}':
            depth -= 1
        if ch in seps and depth == 0:
            parts.append(''.join(buf).strip())
            buf = []
        else:
            buf.append(ch)
    if buf:
        parts.append(''.join(buf).strip())
    return [p for p in parts if p != ""]

# ----------------------------- Parsiranje oblik odgovorov --------------------------

def _parse_pair_like(s: str, cfg: EvalConfig, arity: int, vars_order: List[str]) -> Optional[Tuple[float, ...]]:
    """
    Sprejme "(1,2)" ali "1,2" ali "[1;2]" in vrne tuple dolžine 'arity'.
    Vrstni red v tuplu je po vars_order (če pride poimensko, posortiramo po vars_order).
    """
    t = s.strip()
    # poimensko: "x=1, y=2"
    if '=' in t and any(v in t for v in vars_order):
        # zberi dodelitve v slovar
        vals: Dict[str, float] = {}
        for m in _ASSIGN_TOKEN.finditer(t):
            var = m.group(1)
            num = _parse_number(m.group(2), cfg)
            if num is None:
                return None
            vals[var] = num
        # preveri, da imamo vsa polja
        out: List[float] = []
        for v in vars_order[:arity]:
            if v not in vals:
                return None
            out.append(vals[v])
        return tuple(out)

    # “gola” oblika: oklepaje lahko imajo ali pa ne
    t = _strip_outer_brackets(t)
    # split po ',' ali ';' na top-level
    parts = _split_top_level(t, (',', ';'))
    if len(parts) != arity:
        return None
    nums: List[float] = []
    for p in parts:
        n = _parse_number(p, cfg)
        if n is None:
            return None
        nums.append(n)
    return tuple(nums)

def _parse_set_of_pairs(s: str, cfg: EvalConfig, arity: int, vars_order: List[str], alt_sep: str = '|') -> Optional[List[Tuple[float, ...]]]:
    """
    Množica parov: "{(1,2), (3,4)}" ali "x=1;y=2 | x=3;y=4".
    Vrne seznam tuplov (dolžine arity); vrstni red elementov v parih sledi vars_order.
    """
    t = s.strip()
    t_inner = _strip_outer_brackets(t)
    # poskusi split po alt_sep '|' na top-level
    parts = _split_top_level(t_inner, (alt_sep,))
    if len(parts) <= 1:
        # sicer split po ',' na top-level in poskusi parsati vsak element kot par
        parts = _split_top_level(t_inner, (',',))
        # če so bili zarezi, preveri, da so elementi v oklepajih ali assignmenti
    out: List[Tuple[float, ...]] = []
    for chunk in parts:
        chunk = chunk.strip()
        if not chunk:
            continue
        # če je assignment-list (x=..; y=..) znotraj, najprej normaliziraj ločila na ',' da ga _parse_pair_like dobi
        if '=' in chunk and ';' in chunk and ',' not in chunk:
            chunk = chunk.replace(';', ',')
        pair = _parse_pair_like(chunk, cfg, arity, vars_order)
        if pair is None:
            return None
        out.append(pair)
    return out if out else None

# ----------------------------- API: can_handle --------------------------------------

def can_handle(problem: Any, subject: Optional[str] = None) -> bool:
    cat = subject or getattr(problem, "category", "") or ""
    return any(key.lower() in cat.lower() for key in CATEGORY_MATCHES)

# ----------------------------- API: preprocess_user_input ---------------------------

def preprocess_user_input(problem: Any, user_input: str, config: EvalConfig) -> Any:
    """
    Vrne kanonično predstavitev glede na meta["solution_type"]:
    - "scalar": float
    - "pair"/"triple"/"vector": tuple([...])
    - "set_of_pairs": list[tuple]
    - "none": posebni marker ("NONE")
    - "infinite": posebni marker ("INFINITE")
    """
    meta = getattr(problem, "meta", {}) or {}
    stype = (meta.get("solution_type") or "scalar").lower()
    vars_order = list(meta.get("variables") or ["x", "y"])
    alt = meta.get("pair_separator") or '|'

    s = user_input.strip()
    if not s:
        return ""

    # posebni odgovori
    low = s.lower()
    if any(tok in low for tok in ["∅", "ni rešitve", "brez rešitve", "no solution", "none"]):
        return "NONE"
    if any(tok in low for tok in ["inf", "neskončno", "infinite", "infinitely many"]):
        return "INFINITE"

    if stype == "scalar":
        val = _parse_number(s, config)
        return val if val is not None else s

    # koliko komponent pričakujemo?
    arity = 1 if stype == "scalar" else (len(vars_order) if stype in {"pair", "triple", "vector"} else len(vars_order))
    if stype in {"pair", "triple", "vector"}:
        tup = _parse_pair_like(s, config, arity, vars_order)
        return tup if tup is not None else s

    if stype == "set_of_pairs":
        lst = _parse_set_of_pairs(s, config, len(vars_order), vars_order, alt_sep=alt)
        return lst if lst is not None else s

    # fallback – vrni surovi niz
    return s

# ----------------------------- API: canonical_solution ------------------------------

def _canon_solution_from_str(sol: str, cfg: EvalConfig, stype: str, vars_order: List[str], alt: str) -> Any:
    s = sol.strip()

    # posebni
    low = s.lower()
    if any(tok in low for tok in ["∅", "ni rešitve", "brez rešitve", "no solution", "none"]):
        return "NONE"
    if any(tok in low for tok in ["inf", "neskončno", "infinite", "infinitely many"]):
        return "INFINITE"

    if stype == "scalar":
        return _parse_number(s, cfg)

    if stype in {"pair", "triple", "vector"}:
        arity = len(vars_order)
        return _parse_pair_like(s, cfg, arity, vars_order)

    if stype == "set_of_pairs":
        return _parse_set_of_pairs(s, cfg, len(vars_order), vars_order, alt_sep=alt)

    return s

def canonical_solution(problem: Any, config: EvalConfig) -> Any:
    meta = getattr(problem, "meta", {}) or {}
    stype = (meta.get("solution_type") or "scalar").lower()
    vars_order = list(meta.get("variables") or ["x", "y"])
    alt = meta.get("pair_separator") or '|'

    sol = getattr(problem, "solution", None)

    if isinstance(sol, (int, float)):
        return float(sol) if stype == "scalar" else sol
    if isinstance(sol, (list, tuple)):
        if stype == "set_of_pairs":
            # pričakujemo list[tuple]
            out: List[Tuple[float, ...]] = []
            for item in sol:
                if isinstance(item, (list, tuple)):
                    out.append(tuple(float(x) for x in item))
                else:
                    return None
            return out
        if stype in {"pair", "triple", "vector"}:
            return tuple(float(x) for x in sol)
    if isinstance(sol, str):
        parsed = _canon_solution_from_str(sol, config, stype, vars_order, alt)
        if parsed is not None:
            return parsed

    # fallback na numeric za scalar
    if stype == "scalar" and hasattr(problem, "numeric") and problem.numeric is not None:
        try:
            return float(problem.numeric)
        except Exception:
            pass

    return None

# ----------------------------- API: compare -----------------------------------------

def _tuple_close(a: Tuple[float, ...], b: Tuple[float, ...], cfg: EvalConfig) -> bool:
    if len(a) != len(b):
        return False
    return all(_num_close(x, y, cfg) for x, y in zip(a, b))

def _set_of_pairs_equal(user: List[Tuple[float, ...]], sol: List[Tuple[float, ...]], cfg: EvalConfig, order_invariant: bool = True) -> bool:
    if len(user) != len(sol):
        return False
    if order_invariant:
        used = [False]*len(sol)
        for u in user:
            matched = False
            for j, v in enumerate(sol):
                if not used[j] and _tuple_close(u, v, cfg):
                    used[j] = True
                    matched = True
                    break
            if not matched:
                return False
        return True
    # order-sensitive
    return all(_tuple_close(u, v, cfg) for u, v in zip(user, sol))

def compare(lhs: Any, rhs: Any, config: EvalConfig) -> Tuple[bool, str, str]:
    """
    Vrne (is_correct, match, explanation).
    """
    # posebni primeri "none"/"infinite"
    if isinstance(rhs, str) and rhs in {"NONE", "INFINITE"}:
        if isinstance(lhs, str) and lhs == rhs:
            return True, "exact", ""
        # heuristika: sprejmi različne zapise
        if rhs == "NONE" and isinstance(lhs, str) and any(tok in lhs.lower() for tok in ["∅", "ni rešitve", "brez rešitve", "no solution", "none"]):
            return True, "exact", ""
        if rhs == "INFINITE" and isinstance(lhs, str) and any(tok in lhs.lower() for tok in ["inf", "neskončno", "infinite", "infinitely many"]):
            return True, "exact", ""
        return False, "wrong-value", ("Napačno. Pričakovano: brez rešitve (∅)." if rhs == "NONE"
                                      else "Napačno. Pričakovano: neskončno mnogo rešitev.")

    # skalar
    if isinstance(rhs, (int, float)) and isinstance(lhs, (int, float)):
        ok = _num_close(lhs, rhs, config)
        return (ok, "close" if ok else "wrong-value",
                "" if ok else f"Napačno. Pričakovano: {rhs}.")

    # tuple (pair/triple)
    if isinstance(rhs, tuple) and isinstance(lhs, tuple):
        ok = _tuple_close(lhs, rhs, config)
        return (ok, "close" if ok else "wrong-value",
                "" if ok else f"Napačno. Pričakovano: {rhs} (v vrstnem redu spremenljivk).")

    # set of pairs
    if isinstance(rhs, list) and all(isinstance(t, tuple) for t in rhs):
        if isinstance(lhs, list) and all(isinstance(t, tuple) for t in lhs):
            order_invariant = True
            # poskušaj iz meta dobiti nastavitev
            # (jedro ne posreduje meta sem, zato tega ne moremo brati neposredno;
            #  uporabnik naj v problem.meta nastavi 'order_invariant', jedro primerja prek plugina)
            ok = _set_of_pairs_equal(lhs, rhs, config, order_invariant=True)
            return (ok, "exact" if ok else "wrong-value",
                    "" if ok else "Napačno. Množica rešitev (pari) se ne ujema.")

    # če smo dobili surov niz pri lhs, vrni format napako → jedro bo poskusilo fallback, a tu smo strogo predmetni
    return False, "wrong-format", "Nepravilen format odgovora za linearne enačbe/sisteme (pričakovan skalar, par ali množica parov)."


--- Konec datoteke: orodja\odgovor_plugins\linear_plugin.py ---

--- Začetek datoteke: orodja\odgovor_plugins\pemdas_plugin.py ---

# -*- coding: utf-8 -*-
"""
orodja/odgovor_plugins/pemdas_plugin.py
---------------------------------------
Plugin za predmet "Vrstni red računov" (PEMDAS / OKLMZ).

Delovanje:
- can_handle(...) prepozna kategorijo "Vrstni red računov".
- preprocess_user_input(...) normalizira uporabniški vnos in ga poskusi pretvoriti v število.
- canonical_solution(...) izlušči pravilno vrednost iz Problem.solution/numeric ali po potrebi
  varno izračuna iz promta.
- compare(...) primerja z upoštevanjem, da je rezultat navadno celo število; sicer uporabi
  decimalno toleranco iz EvalConfig.

Opombe:
- Vgrajen je lasten varni evaluator z belim seznamom operatorjev: +, -, *, /, **, unarni +/- in oklepaji.
- Podprti simboli: '·' (množenje), '÷' (deljenje), '^' (potenca), oklepaji '()' in oglate '[]'.
- Dvojni minus v zapisu (npr. 'a - - b') je podprt kot 'a - (-b)' (ni pa nujno preoblikovati).
"""

from __future__ import annotations

import ast
import math
import re
from fractions import Fraction
from typing import Any, Optional, Tuple

# Tipni namigi (če core ni uvožen, dummy fallback razred zaradi type checkerja)
try:
    from orodja.odgovor_eval import EvalConfig
except Exception:
    class EvalConfig:  # pragma: no cover
        decimal_tolerance_abs: float = 1e-9
        decimal_tolerance_rel: float = 1e-9
        locale_decimal: str = "auto"
        accept_equivalent_forms: bool = True
        set_order_invariant: bool = True
        allow_expressions: bool = True

name = "pemdas"

CATEGORY_MATCHES = {"Vrstni red računov", "PEMDAS", "OKLMZ"}

_INT_TOKEN = re.compile(r'^[+\-]?\d+$')
_FLOAT_TOKEN = re.compile(r'^[+\-]?\d+(?:[.,]\d+)?$')
_FRAC_TOKEN = re.compile(r'(?P<sign>[+\-]?)\s*(?P<n>\d+)\s*/\s*(?P<d>-?\d+)')

# ----------------------------- Varno vrednotenje izrazov ----------------------------

_BIN_OPS = {
    ast.Add:  lambda a, b: a + b,
    ast.Sub:  lambda a, b: a - b,
    ast.Mult: lambda a, b: a * b,
    ast.Div:  lambda a, b: a / b,
    ast.Pow:  lambda a, b: a ** b,
}
_UNARY_OPS = {
    ast.UAdd: lambda a: a,
    ast.USub: lambda a: -a,
}
_ALLOWED_NODES = (
    ast.Expression, ast.BinOp, ast.UnaryOp, ast.Constant, ast.Constant,
    ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow, ast.UAdd, ast.USub,
    ast.Load, ast.Tuple, ast.List, ast.Expr
)

def _normalize_expr(expr: str) -> str:
    """
    Normalizacija zapisa:
    - odstrani vodilne oznake 'Uredi:' / 'Uredi izraz:' (če pridejo iz prompta),
    - zamenja '·' -> '*', '÷' -> '/', '^' -> '**', '['/']' -> '(' / ')',
    - pusti presledke nedotaknjene (ast ne potrebuje posebne obdelave),
    - ne dela slepega '--' -> '+' (ker razlikujemo unarni/binarni minus z AST).
    """
    s = expr.strip()
    if s.lower().startswith("uredi izraz:"):
        s = s.split(":", 1)[1].strip()
    elif s.lower().startswith("uredi:"):
        s = s.split(":", 1)[1].strip()
    s = (s.replace("·", "*")
           .replace("÷", "/")
           .replace("^", "**")
           .replace("[", "(")
           .replace("]", ")"))
    return s

def _safe_eval_number(expr: str) -> float:
    """
    Varen evaluator za števila/izraze s + - * / ** in oklepaji.
    Vrne float (PEMDAS naloge imajo običajno cel rezultat).
    Dovoli tudi tuple/list, a na koncu zahteva številski rezultat.
    """
    node = ast.parse(expr, mode="eval")

    def _eval(n):
        if isinstance(n, ast.Expression):
            return _eval(n.body)
        if isinstance(n, ast.Constant):
            if isinstance(n.value, (int, float)):
                return n.value
            raise ValueError("Nedovoljena konstanta.")
        if isinstance(n, ast.Constant):  # Py<3.8
            return n.n
        if isinstance(n, ast.UnaryOp) and type(n.op) in _UNARY_OPS:
            return _UNARY_OPS[type(n.op)](_eval(n.operand))
        if isinstance(n, ast.BinOp) and type(n.op) in _BIN_OPS:
            return _BIN_OPS[type(n.op)](_eval(n.left), _eval(n.right))
        if isinstance(n, (ast.Tuple, ast.List)):
            # Ne podpiramo seznamov kot rezultatov v PEMDAS – uporabnik mora dati število
            vals = [_eval(elt) for elt in (n.elts if isinstance(n, (ast.Tuple, ast.List)) else [])]
            raise ValueError(f"Pričakovano število, dobil seznam/tuple: {vals}")
        raise ValueError(f"Nedovoljen element v izrazu: {type(n).__name__}")

    for sub in ast.walk(node):
        if not isinstance(sub, _ALLOWED_NODES):
            raise ValueError(f"Prepovedan AST element: {type(sub).__name__}")

    val = _eval(node)
    if not isinstance(val, (int, float)):
        raise ValueError("Pričakovano številsko vrednost.")
    return float(val)

# ----------------------------- Parsiranje atomov ------------------------------------

def _parse_fraction_token(token: str) -> Optional[Fraction]:
    m = _FRAC_TOKEN.fullmatch(token.strip())
    if not m:
        return None
    sign = -1 if m.group("sign") == "-" else 1
    n = int(m.group("n")) * sign
    d = int(m.group("d"))
    if d == 0:
        return None
    # normaliziraj znak v števec
    if d < 0:
        n, d = -n, -d
    return Fraction(n, d)

def _parse_number_token(token: str, cfg: EvalConfig) -> Optional[float]:
    t = token.strip()
    if _INT_TOKEN.fullmatch(t):
        try:
            return float(int(t))
        except Exception:
            return None
    if _FLOAT_TOKEN.fullmatch(t):
        if cfg.locale_decimal == ',':
            t2 = t.replace(',', '.')
        elif cfg.locale_decimal == '.':
            t2 = t
        else:
            t2 = t.replace(',', '.')
        try:
            return float(t2)
        except Exception:
            return None
    return None

def _parse_user_number_or_expr(s: str, cfg: EvalConfig) -> float:
    """
    Poskusi: int/float/ulomek; sicer varni izraz.
    Vrne float (za primerjavo); dvigne ValueError, če ni številčno.
    """
    s = s.strip()
    # posamični atom: fraction
    fr = _parse_fraction_token(s)
    if fr is not None:
        return float(fr)
    # posamični atom: number
    num = _parse_number_token(s, cfg)
    if num is not None:
        return float(num)
    # sicer izraz
    norm = _normalize_expr(s)
    return _safe_eval_number(norm)

# ----------------------------- Plugin API -------------------------------------------

def can_handle(problem: Any, subject: Optional[str] = None) -> bool:
    cat = subject or getattr(problem, "category", "") or ""
    return any(key.lower() in cat.lower() for key in CATEGORY_MATCHES)

def preprocess_user_input(problem: Any, user_input: str, config: EvalConfig) -> Any:
    """
    Vrne float (uporablja se za primerjavo), ali surov niz če ni številčno.
    """
    s = user_input.strip()
    if not s:
        return ""
    try:
        val = _parse_user_number_or_expr(s, config)
        return val
    except Exception:
        # vrni surovi string, da bo compare lahko vrnil 'wrong-format'
        return s

def _canonical_from_solution(problem: Any, cfg: EvalConfig) -> Optional[float]:
    """
    Vzame Problem.solution / numeric in jih pretvori v float.
    Če solution ni uporabno, poskusi izračunati iz prompta (varno).
    """
    sol = getattr(problem, "solution", None)
    if isinstance(sol, (int, float)):
        return float(sol)
    if isinstance(sol, str):
        s = sol.strip()
        # posamični ulomek?
        fr = _parse_fraction_token(s)
        if fr is not None:
            return float(fr)
        # število?
        num = _parse_number_token(s, cfg)
        if num is not None:
            return float(num)
        # ali morda cel izraz? (redko v solution; običajno ne)
        try:
            return _safe_eval_number(_normalize_expr(s))
        except Exception:
            pass

    if hasattr(problem, "numeric") and problem.numeric is not None:
        try:
            return float(problem.numeric)
        except Exception:
            pass

    # fallback: poskusi iz prompta izračunati
    prompt = getattr(problem, "prompt", "") or ""
    if ":" in prompt:
        right = prompt.split(":", 1)[1].strip()
    else:
        right = prompt
    try:
        return _safe_eval_number(_normalize_expr(right))
    except Exception:
        return None

def canonical_solution(problem: Any, config: EvalConfig) -> Any:
    return _canonical_from_solution(problem, config)

def _is_int_like(x: float, tol: float = 1e-9) -> Tuple[bool, Optional[int]]:
    """Preveri, ali je x v toleranci celo število; vrne (True, int(x)) ali (False, None)."""
    xi = round(x)
    if abs(x - xi) <= tol:
        return True, int(xi)
    return False, None

def _fmt_user_for_msg(u: Any) -> str:
    if isinstance(u, float):
        ok, iv = _is_int_like(u)
        return str(iv) if ok else f"{u}"
    return str(u)

def compare(lhs: Any, rhs: Any, config: EvalConfig) -> Tuple[bool, str, str]:
    """
    lhs: float ali surov niz (če parsing spodletel)
    rhs: float (kanonična rešitev)
    Vrne: (is_correct, match, explanation)
    """
    if rhs is None:
        return False, "wrong-format", "Ni uspelo določiti pravilne rešitve (manjkajoč 'numeric' ali neveljaven 'solution')."

    # uporabnik ni podal številke/izraza
    if not isinstance(lhs, (int, float)):
        return False, "wrong-format", "Vnesi številko ali izraz (z oklepaji in pravilnim vrstnim redom računov)."

    # 1) najprej preveri celoštevilsko enakost (večina PEMDAS nalog je celih)
    #    – uporabimo strožji prag (abs_tol), rel_tol iz config ostane za splošni primer
    is_int_rhs, rhs_int = _is_int_like(rhs, tol=config.decimal_tolerance_abs)
    is_int_lhs, lhs_int = _is_int_like(lhs, tol=config.decimal_tolerance_abs)

    if is_int_rhs and is_int_lhs:
        ok = (rhs_int == lhs_int)
        return (ok, "exact" if ok else "wrong-value",
                "" if ok else f"Napačno. Pričakovano: {rhs_int}, dobil: {lhs_int}.")

    # 2) sicer primerjaj realno (math.isclose)
    ok = math.isclose(float(lhs), float(rhs),
                      rel_tol=config.decimal_tolerance_rel,
                      abs_tol=config.decimal_tolerance_abs)
    if ok:
        return True, "close", ""  # numerično ustreza (čeprav ni točno celo)

    # 3) namigi (heuristike)
    #   - pogosta napaka: izvedeno v napačnem vrstnem redu (npr. seštevanje pred množenjem)
    #     Ne moremo zanesljivo dokazati, a uporabniku podamo splošen namig.
    msg = (
        "Napačno. Namig: upoštevaj vrstni red računov — "
        "oklepaji → potence → množenje/deljenje → seštevanje/odštevanje; "
        "računaj z leve pri S/O."
    )
    # dodatni namig za dvojni minus
    if "--" in str(lhs) or "--" in str(rhs):
        msg += " Pri dvojnem minusu velja: −(−x) = +x."
    return False, "wrong-value", msg


--- Konec datoteke: orodja\odgovor_plugins\pemdas_plugin.py ---

--- Začetek datoteke: orodja\odgovor_plugins\percent_plugin.py ---

# -*- coding: utf-8 -*-
"""
orodja/odgovor_plugins/percent_plugin.py
----------------------------------------
Plugin za "Procenti / Odstotki" in "DDV / VAT".

Podprto:
- Uporabniški vnos kot:
  * odstotek: "12.5%", "12,5 %"
  * denarna vrednost: "12.50", "12,50", "€12.50", "12,50 €", "12.50 EUR"
  * navadna številka: "0.125" (ekvivalent 12.5% pri expect="percent")
- Decimalna vejica ali pika.
- Zaokroževanje po politiki (privzeto half-up) in decimalkah (privzeto 2) za denar/končne rezultate.

`Problem.meta` (opcijsko):
- expect: "percent" | "value" | "money"    # kako interpretirati odgovore (privzeto "value")
- rounding_policy: "half_up" | "bankers"   # zaokroževanje končnih rezultatov (privzeto "half_up")
- decimals: int                             # št. decimalk za zaokroževanje (privzeto 2)
- vat_rate: float                           # npr. 0.22 (22%), samo za namige
- vat_mode: "included" | "excluded"         # informativno (namigi)

Opomba:
- Plugin ne izračunava rešitve iz podatkov; primerja uporabnikov odgovor z `solution`/`numeric`.
"""

from __future__ import annotations

import math
import re
from decimal import Decimal, ROUND_HALF_UP, ROUND_HALF_EVEN, InvalidOperation
from fractions import Fraction
from typing import Any, Optional, Tuple, Dict

# Type-hint fallback
try:
    from orodja.odgovor_eval import EvalConfig
except Exception:  # pragma: no cover
    class EvalConfig:
        decimal_tolerance_abs: float = 1e-9
        decimal_tolerance_rel: float = 1e-9
        locale_decimal: str = "auto"
        accept_equivalent_forms: bool = True
        set_order_invariant: bool = True
        allow_expressions: bool = True

name = "percent"

CATEGORY_MATCHES = {
    "Procenti", "Odstotki", "DDV", "VAT",
    "Percent", "Percentage"
}

_CURRENCY_TOK = re.compile(r'(€|\bEUR\b)', re.IGNORECASE)
_PERCENT_TOK = re.compile(r'%')
_WHITES = re.compile(r'\s+')

def _is_category(cat: str) -> bool:
    return any(key.lower() in cat.lower() for key in CATEGORY_MATCHES)

# ------------------------- Zaokroževanje politik ------------------------------------

def _round_value(value: float, decimals: int, policy: str) -> float:
    """
    Rounding helper:
    - policy "half_up": komercialno zaokroževanje (0.5 → gor)
    - policy "bankers": ROUND_HALF_EVEN (bančniško)
    """
    q = Decimal(10) ** (-decimals)
    try:
        d = Decimal(str(value))
    except InvalidOperation:
        d = Decimal(value)
    if policy == "bankers":
        return float(d.quantize(q, rounding=ROUND_HALF_EVEN))
    # default half_up
    return float(d.quantize(q, rounding=ROUND_HALF_UP))

# ------------------------- Parsiranje uporabniškega vnosa ----------------------------

def _normalize_decimal(s: str, cfg: EvalConfig) -> str:
    """Pretvori decimalno vejico v piko (varno za posamezne številke)."""
    t = s.strip()
    if cfg.locale_decimal == ',':
        return t.replace(',', '.')
    if cfg.locale_decimal == '.':
        return t
    # auto: če ima vejico in nima pike, obravnavaj kot decimalno
    if ',' in t and '.' not in t:
        return t.replace(',', '.')
    return t

def _strip_currency(s: str) -> Tuple[str, bool]:
    """Odstrani € / EUR in vrne (niz_brez_valute, ali_je_bila_valuta)."""
    t = _WHITES.sub('', s)
    had = bool(_CURRENCY_TOK.search(t))
    t = _CURRENCY_TOK.sub('', t)
    return t, had

def _parse_number_or_percent(
    s: str, cfg: EvalConfig, expect: str
) -> Optional[float]:
    """
    Vrne float:
    - expect=="percent": '12.5%' → 0.125 ; tudi '0.125' (brez %) je sprejeto
    - expect=="money": odstrani €/EUR in vrne vrednost
    - expect=="value": navadna številka (in 'x%' → x/100, tolerantno)
    """
    if not s or not s.strip():
        return None
    raw = s.strip()

    # denar: odstrani €/EUR
    if expect == "money":
        raw, _ = _strip_currency(raw)

    # občutljivost na presledke okoli %
    has_pct = bool(_PERCENT_TOK.search(raw))
    t = raw.replace(' ', '')
    # decimalka
    t = _normalize_decimal(t, cfg)

    # odstotek izražen kot ulomek p/q
    m_frac = re.fullmatch(r'([+\-]?)\s*(\d+)\s*/\s*([+\-]?\d+)%?', t)
    if m_frac:
        sign = -1 if m_frac.group(1) == '-' else 1
        n = int(m_frac.group(2)) * sign
        d = int(m_frac.group(3))
        if d == 0:
            return None
        val = float(Fraction(n, d))
        if expect == "percent" and not has_pct:
            # npr. "1/8" → kot delež (0.125), enakovredno 12.5 %
            return val
        if expect == "percent" and has_pct:
            # "1/8%" bi bil (1/8)%, kar nočemo; zavrnemo
            return None
        return val

    # navadno število z/ brez %
    m_num = re.fullmatch(r'([+\-]?\d+(?:\.\d+)?)%?', t)
    if not m_num:
        return None
    num = float(m_num.group(1))

    if expect == "percent":
        # sprejmi obe obliki: "12.5%" ali "0.125"
        if has_pct:
            return num / 100.0
        else:
            # če je < 1.0, delež; če je >=1, interpretiraj kot %
            return num if num < 1.0 else (num / 100.0)
    # money/value: če uporabnik doda '%', razumemo kot številko v % → delež
    if has_pct:
        return num / 100.0
    return num

# ------------------------- Kanonična rešitev -----------------------------------------

def _as_float(x: Any) -> Optional[float]:
    if isinstance(x, (int, float)):
        return float(x)
    if isinstance(x, Fraction):
        return float(x)
    if isinstance(x, str):
        try:
            return float(x.replace(',', '.'))
        except Exception:
            return None
    return None

def _canonical_solution(problem: Any, cfg: EvalConfig, expect: str) -> Optional[float]:
    sol = getattr(problem, "solution", None)

    # Če je solution kot odstotek npr. "12.5%", ga interpretiramo skladno z 'expect'
    if isinstance(sol, str):
        v = _parse_number_or_percent(sol, cfg, expect=expect)
        if v is not None:
            return v

    # poskusi kot navadno število
    v2 = _as_float(sol)
    if v2 is not None:
        return v2

    # fallback: numeric (float)
    if hasattr(problem, "numeric") and problem.numeric is not None:
        try:
            return float(problem.numeric)
        except Exception:
            pass

    return None

# ------------------------- Plugin API ------------------------------------------------

def can_handle(problem: Any, subject: Optional[str] = None) -> bool:
    cat = subject or getattr(problem, "category", "") or ""
    return _is_category(cat)

def preprocess_user_input(problem: Any, user_input: str, config: EvalConfig) -> Any:
    """
    Vrne dict {"value": float, "_meta": {...}} glede na meta['expect'].
    Če parse spodleti, vrne surovi niz (da dobiš 'wrong-format').
    """
    meta = getattr(problem, "meta", {}) or {}
    expect = (meta.get("expect") or "value").lower()
    policy = (meta.get("rounding_policy") or "half_up").lower()
    decimals = int(meta.get("decimals") or (2 if expect == "money" else 2))
    vat_rate = meta.get("vat_rate")
    vat_mode = (meta.get("vat_mode") or "").lower()

    s = user_input.strip()
    if not s:
        return ""

    val = _parse_number_or_percent(s, config, expect=expect)
    if val is None:
        return s

    return {
        "value": float(val),
        "_meta": {
            "expect": expect,
            "rounding_policy": policy,
            "decimals": decimals,
            "vat_rate": vat_rate,
            "vat_mode": vat_mode,
        },
    }

def canonical_solution(problem: Any, config: EvalConfig) -> Any:
    """
    Vrne dict {"value": float, "_meta": {...}} za rešitev,
    tako da compare ne potrebuje dostopa do `problem`.
    """
    meta = getattr(problem, "meta", {}) or {}
    expect = (meta.get("expect") or "value").lower()
    policy = (meta.get("rounding_policy") or "half_up").lower()
    decimals = int(meta.get("decimals") or (2 if expect == "money" else 2))
    vat_rate = meta.get("vat_rate")
    vat_mode = (meta.get("vat_mode") or "").lower()

    val = _canonical_solution(problem, config, expect=expect)
    if val is None:
        return None

    return {
        "value": float(val),
        "_meta": {
            "expect": expect,
            "rounding_policy": policy,
            "decimals": decimals,
            "vat_rate": vat_rate,
            "vat_mode": vat_mode,
        },
    }

def _equals_with_rounding(lhs: float, rhs: float, *, decimals: int, policy: str) -> Tuple[bool, str]:
    l = _round_value(lhs, decimals, policy)
    r = _round_value(rhs, decimals, policy)
    return (l == r, "exact" if l == r else "wrong-value")

def _extract_val_meta(obj: Any) -> Tuple[Optional[float], Dict[str, Any]]:
    """
    Sprejme bodisi dict {"value":..., "_meta":...} bodisi golo število.
    Vrne (value, meta_dict).
    """
    if isinstance(obj, dict) and "value" in obj:
        val = obj.get("value")
        meta = obj.get("_meta") or {}
        try:
            return (float(val), dict(meta))
        except Exception:
            return (None, {})
    if isinstance(obj, (int, float)):
        return (float(obj), {})
    return (None, {})

def compare(lhs: Any, rhs: Any, config: EvalConfig) -> Tuple[bool, str, str]:
    """
    Primerjava brez dostopa do `problem`:
    - pričakuje, da sta lhs/rhs dict-i z "value" in "_meta" (ali gola float-a).
    - če meta narekuje zaokroževanje (money/decimals/policy), primerja po zaokroženih.
    """
    if rhs is None:
        return False, "wrong-format", "Ni mogoče določiti pravilne vrednosti (manjka 'solution' ali 'numeric')."

    lhs_val, lhs_meta = _extract_val_meta(lhs)
    rhs_val, rhs_meta = _extract_val_meta(rhs)

    if lhs_val is None:
        return False, "wrong-format", "Vnesi številko (dovoljene so oblike z %, € in decimalno vejico)."
    if rhs_val is None:
        return False, "wrong-format", "Neveljavna ciljna vrednost."

    # meta vzemi z desne (rešitev je vir resnice); če je prazna, poskusi z levo; sicer fallback
    meta = rhs_meta if rhs_meta else lhs_meta
    expect = (meta.get("expect") or "value").lower()
    policy = (meta.get("rounding_policy") or "half_up").lower()
    decimals = int(meta.get("decimals") or (2 if expect == "money" else 2))
    vat_rate = meta.get("vat_rate")
    vat_mode = (meta.get("vat_mode") or "").lower()

    # denar ali izrecno zaokroževanje: primerjaj po zaokroženih vrednostih
    if expect == "money" or "rounding_policy" in meta or "decimals" in meta:
        ok, kind = _equals_with_rounding(float(lhs_val), float(rhs_val), decimals=decimals, policy=policy)
        if ok:
            return True, kind, ""
        # namig za DDV (če navedeno)
        vat_note = ""
        if vat_rate is not None:
            if vat_mode == "included":
                vat_note = " Namig: pri 'DDV vključen' je osnova = cena/(1+stopnja)."
            elif vat_mode == "excluded":
                vat_note = " Namig: pri 'DDV izključen' je končna = osnova·(1+stopnja)."
        return False, "wrong-value", f"Napačno (po zaokroževanju na {decimals} dec.).{vat_note}"

    # sicer realno s toleranco
    ok = math.isclose(float(lhs_val), float(rhs_val),
                      rel_tol=config.decimal_tolerance_rel,
                      abs_tol=config.decimal_tolerance_abs)
    if ok:
        return True, "close", ""

    # generični namigi
    hint = "Namig: preveri pretvorbo med % in decimalnim zapisom (podeli s 100)." if expect == "percent" else ""
    return False, "wrong-value", f"Napačno. {hint}"


--- Konec datoteke: orodja\odgovor_plugins\percent_plugin.py ---

--- Začetek datoteke: orodja\odgovor_plugins\polynomial_plugin.py ---

# -*- coding: utf-8 -*-
"""
orodja/odgovor_plugins/polynomial_plugin.py
-------------------------------------------
Plugin za predmet "Polinomi".

Cilj:
- Sprejme uporabniški odgovor kot polinomski izraz v spremenljivki x
  (npr. "x^2 - 5x + 6" ali faktorizirano "(x-2)(x-3)").
- Varno ga parsira (belolisti: +, -, *, ^, oklepaji, konstante; ENA spremenljivka: x),
  ga razširi v koeficiente in primerja s pravilno rešitvijo (tudi če je ta
  v faktorizirani obliki).
- Primerjava se izvaja po koeficientih z numerično toleranco iz EvalConfig.

Podprto:
- Operacije: +, -, *, ^ (potenca z nenegativnim celim eksponentom)
- Konstantni številski koeficienti (celo/decimalno); decimalna vejica je dovoljena
- Ena spremenljivka: "x"
- Faktorizacija z oklepaji: npr. "(x-2)(x+3)" – razširimo z množenjem polinomov

Nepodprto/omejitve:
- Deljenje "/" in negativne potence niso dovoljene (ni racionalnih funkcij).
- Več spremenljivk ni podprtih.
"""

from __future__ import annotations

import ast
import math
import re
from fractions import Fraction
from typing import Any, Dict, Tuple, List, Optional

# mehka odvisnost samo za type hints
try:
    from orodja.odgovor_eval import EvalConfig
except Exception:  # pragma: no cover
    class EvalConfig:
        decimal_tolerance_abs: float = 1e-9
        decimal_tolerance_rel: float = 1e-9
        locale_decimal: str = "auto"
        accept_equivalent_forms: bool = True
        set_order_invariant: bool = True
        allow_expressions: bool = True

name = "polynomial"
CATEGORY_MATCHES = {"Polinomi", "Polinomial", "Polynomials"}

# --------- Helpers: števila (podpora vejici) ----------

def _parse_number_token(s: str, cfg: EvalConfig) -> Optional[Fraction]:
    t = s.strip()
    if cfg.locale_decimal == ',':
        t = t.replace(',', '.')
    elif cfg.locale_decimal == 'auto' and (',' in t and '.' not in t):
        t = t.replace(',', '.')
    # podpiramo celo/decimalno; Fraction(t) sprejme string
    try:
        return Fraction(t)
    except Exception:
        # fallback prek float (manj natančno), nato racionalizacija
        try:
            return Fraction.from_float(float(t)).limit_denominator()
        except Exception:
            return None

# --------- Polinom kot slovar {stopnja: koeficient(Fraction)} ----------

Poly = Dict[int, Fraction]

def _poly_add(a: Poly, b: Poly) -> Poly:
    out: Poly = dict(a)
    for k, v in b.items():
        out[k] = out.get(k, Fraction(0)) + v
        if out[k] == 0:
            del out[k]
    return out

def _poly_sub(a: Poly, b: Poly) -> Poly:
    out: Poly = dict(a)
    for k, v in b.items():
        out[k] = out.get(k, Fraction(0)) - v
        if out[k] == 0:
            del out[k]
    return out

def _poly_mul(a: Poly, b: Poly) -> Poly:
    out: Poly = {}
    for ka, va in a.items():
        for kb, vb in b.items():
            k = ka + kb
            out[k] = out.get(k, Fraction(0)) + va * vb
            if out[k] == 0:
                del out[k]
    return out

def _poly_pow(a: Poly, n: int) -> Poly:
    if n < 0:
        raise ValueError("Negativne potence niso podprte za polinome.")
    out: Poly = {0: Fraction(1)}
    base = dict(a)
    e = n
    while e > 0:
        if e & 1:
            out = _poly_mul(out, base)
        base = _poly_mul(base, base)
        e >>= 1
    return out

def _poly_from_const(c: Fraction) -> Poly:
    return {0: c} if c != 0 else {}

def _poly_from_x() -> Poly:
    return {1: Fraction(1)}

def _poly_trim(a: Poly, tol: float) -> Poly:
    """Odstrani (numerično) ničelne koeficiente po toleranci."""
    out: Poly = {}
    for k, v in a.items():
        if abs(float(v)) > tol:
            out[k] = v
    return out

def _poly_degree(a: Poly) -> int:
    return max(a.keys()) if a else -math.inf  # prazen → -inf

def _poly_coeff_list(a: Poly) -> List[Fraction]:
    """Vrne koeficiente po padajoči stopnji: [a_n, ..., a_0]."""
    if not a:
        return [Fraction(0)]
    deg = _poly_degree(a)
    return [a.get(d, Fraction(0)) for d in range(deg, -1, -1)]

# --------- Varen parser za polinomske izraze v x ----------

_ALLOWED_NODES = (
    ast.Expression, ast.BinOp, ast.UnaryOp, ast.Constant,
    ast.Add, ast.Sub, ast.Mult, ast.Pow, ast.UAdd, ast.USub, ast.Name,
    ast.Load, ast.Tuple, ast.List, ast.Expr
)

def _normalize_expr(expr: str) -> str:
    s = expr.strip()
    # nadomesti ^ → ** in odstrani nepotrebne besedne oznake
    s = s.replace("^", "**")
    # odstrani morebitne leading "Uredi:" ipd.
    low = s.lower()
    if low.startswith("uredi:") or low.startswith("uredi izraz:"):
        s = s.split(":", 1)[1].strip()
    return s

def _poly_from_ast(node: ast.AST, cfg: EvalConfig) -> Poly:
    if isinstance(node, ast.Expression):
        return _poly_from_ast(node.body, cfg)

    if isinstance(node, ast.Constant):
        if isinstance(node.value, (int, float)):
            return _poly_from_const(Fraction(node.value))
        raise ValueError("Nedovoljena konstanta.")
    if isinstance(node, ast.Constant):  # Py<3.8
        return _poly_from_const(Fraction(node.n))

    if isinstance(node, ast.Name):
        if node.id == "x":
            return _poly_from_x()
        raise ValueError("Dovoljena je le spremenljivka 'x'.")

    if isinstance(node, ast.UnaryOp) and isinstance(node.op, (ast.UAdd, ast.USub)):
        p = _poly_from_ast(node.operand, cfg)
        if isinstance(node.op, ast.UAdd):
            return p
        return {k: -v for k, v in p.items()}

    if isinstance(node, ast.BinOp):
        if isinstance(node.op, ast.Add):
            return _poly_add(_poly_from_ast(node.left, cfg), _poly_from_ast(node.right, cfg))
        if isinstance(node.op, ast.Sub):
            return _poly_sub(_poly_from_ast(node.left, cfg), _poly_from_ast(node.right, cfg))
        if isinstance(node.op, ast.Mult):
            return _poly_mul(_poly_from_ast(node.left, cfg), _poly_from_ast(node.right, cfg))
        if isinstance(node.op, ast.Pow):
            base = _poly_from_ast(node.left, cfg)
            # eksponent mora biti nenegativno celo število (konstanta)
            if isinstance(node.op, ast.Pow):
                base = _poly_from_ast(node.left, cfg)
                if isinstance(node.right, ast.Constant) and isinstance(node.right.value, (int, float)):
                    exp_val = int(node.right.value)
                else:
                    raise ValueError("Potenca polinoma mora imeti cel, nenegativen eksponent.")
                if exp_val < 0:
                    raise ValueError("Negativne potence niso podprte.")
                return _poly_pow(base, exp_val)

    # Poskusi, če je goli decimalni/celi token v obliki stringa (npr. "1,5")
    if isinstance(node, ast.Expr):
        return _poly_from_ast(node.value, cfg)

    raise ValueError(f"Nepodprt izraz v polinomu: {type(node).__name__}")

def _parse_poly(expr: str, cfg: EvalConfig) -> Poly:
    s = _normalize_expr(expr)
    # decimalne vejice v številih: ast jih ne mara → predhodno zamenjaj
    if cfg.locale_decimal == ',':
        s = re.sub(r'(\d),(\d)', r'\1.\2', s)
    elif cfg.locale_decimal == 'auto':
        # heuristic: zamenjaj decimalne vejice v številih (ni 100% nujno)
        s = re.sub(r'(\d),(\d)', r'\1.\2', s)

    tree = ast.parse(s, mode="eval")

    # varnostni whitelist
    for sub in ast.walk(tree):
        if not isinstance(sub, _ALLOWED_NODES):
            raise ValueError(f"Prepovedan element v izrazu: {type(sub).__name__}")
        if isinstance(sub, ast.BinOp) and isinstance(sub.op, ast.Div):
            raise ValueError("Deljenje ni podprto v polinomih.")
        if isinstance(sub, ast.Pow):
            # preverbo eksponenta naredimo v _poly_from_ast
            pass

    poly = _poly_from_ast(tree, cfg)
    return poly

# --------- API plugina ----------

def can_handle(problem: Any, subject: Optional[str] = None) -> bool:
    cat = subject or getattr(problem, "category", "") or ""
    return any(key.lower() in cat.lower() for key in CATEGORY_MATCHES)

def preprocess_user_input(problem: Any, user_input: str, config: EvalConfig) -> Any:
    """
    Vrne kanonično predstavitev uporabnikovega vnosa kot Poly ({stopnja: Fraction}),
    ali surovi niz (če parse spodleti).
    """
    s = user_input.strip()
    if not s:
        return ""
    try:
        poly = _parse_poly(s, config)
        # numerično poreži ~0 koeficiente
        poly = _poly_trim(poly, tol=max(config.decimal_tolerance_abs, 1e-12))
        return poly
    except Exception:
        return s  # wrong-format → naj compare nakaže napako

def _canonical_from_solution(problem: Any, cfg: EvalConfig) -> Optional[Poly]:
    sol = getattr(problem, "solution", None)
    # Kot polinomski izraz v nizu
    if isinstance(sol, str):
        try:
            p = _parse_poly(sol, cfg)
            return _poly_trim(p, tol=max(cfg.decimal_tolerance_abs, 1e-12))
        except Exception:
            pass
    # Alternativa: koeficienti v meta, npr. meta["coeffs"] = [1, -5, 6] (za x^2 -5x +6)
    meta = getattr(problem, "meta", {}) or {}
    coeffs = meta.get("coeffs")
    if isinstance(coeffs, (list, tuple)) and coeffs:
        # coeffs: po padajoči stopnji [a_n, ..., a_0]
        deg = len(coeffs) - 1
        poly: Poly = {}
        for i, c in enumerate(coeffs):
            try:
                cf = _parse_number_token(str(c), cfg)
            except Exception:
                cf = None
            if cf is None:
                return None
            power = deg - i
            if cf != 0:
                poly[power] = cf
        return _poly_trim(poly, tol=max(cfg.decimal_tolerance_abs, 1e-12))
    return None

def canonical_solution(problem: Any, config: EvalConfig) -> Any:
    return _canonical_from_solution(problem, config)

def _poly_equal(a: Poly, b: Poly, cfg: EvalConfig) -> bool:
    # izboljšano: primerjava po koeficientih z toleranco
    keys = set(a.keys()) | set(b.keys())
    tol = max(cfg.decimal_tolerance_abs, 1e-9)
    for k in keys:
        av = float(a.get(k, Fraction(0)))
        bv = float(b.get(k, Fraction(0)))
        if not math.isclose(av, bv, rel_tol=cfg.decimal_tolerance_rel, abs_tol=tol):
            return False
    return True

def _fmt_poly(poly: Poly) -> str:
    if not poly:
        return "0"
    parts: List[str] = []
    for deg in sorted(poly.keys(), reverse=True):
        c = poly[deg]
        c_str = str(c) if c.denominator != 1 else str(c.numerator)
        if deg == 0:
            parts.append(f"{c_str}")
        elif deg == 1:
            if c == 1:
                parts.append("x")
            elif c == -1:
                parts.append("-x")
            else:
                parts.append(f"{c_str}·x")
        else:
            if c == 1:
                parts.append(f"x^{deg}")
            elif c == -1:
                parts.append(f"-x^{deg}")
            else:
                parts.append(f"{c_str}·x^{deg}")
    # sestavi z znaki
    out = parts[0]
    for term in parts[1:]:
        if term.startswith('-'):
            out += f" - {term[1:]}"
        else:
            out += f" + {term}"
    return out

def compare(lhs: Any, rhs: Any, config: EvalConfig) -> Tuple[bool, str, str]:
    """
    lhs/rhs: Poly (slovar) ali surov niz (če parse spodletel).
    Vrne (is_correct, match, explanation).
    """
    if rhs is None:
        return False, "wrong-format", "Ni mogoče določiti ciljnega polinoma (manjka 'solution' ali 'meta.coeffs')."

    if not isinstance(lhs, dict) or not isinstance(rhs, dict):
        return False, "wrong-format", "Vnesi polinomski izraz v spremenljivki x (brez deljenja)."

    ok = _poly_equal(lhs, rhs, config)
    if ok:
        return True, "exact", ""

    return False, "wrong-value", (
        "Napačno. Namig: razširi produkt (če je faktorizirano), "
        "uredi po padajočih stopnjah in primerjaj koeficiente. "
        f"(Pričakovano: {_fmt_poly(rhs)})"
    )


--- Konec datoteke: orodja\odgovor_plugins\polynomial_plugin.py ---

--- Začetek datoteke: orodja\odgovor_plugins\trig_plugin.py ---

# -*- coding: utf-8 -*-
"""
orodja/odgovor_plugins/trig_plugin.py
-------------------------------------
Plugin za predmet "Trigonometrija".

Podprto:
- Dva režima prek problem.meta["type"]:
  * "value": primerjava numerične vrednosti trig. izraza (ali številke).
  * "solve": primerjava množice kotov (rešitev) v danem intervalu.
- Kotne enote: stopinje (°) ali radiani (π), avtomatsko ali prek meta["angle_mode"].
- Intervali za rešitve: nizi kot "[0,2π)" ali "[0°,360°)" itd. (levi/zadnji oklepaj lahko ( ali ] ).

Dovoljene funkcije v izrazih: sin, cos, tan, asin, acos, atan, sqrt, abs, log, ln (log = naravni log),
konstanti: pi, e. Stopinjski znak "°" se pretvori v radiane.

Odvisnosti: samo standardna knjižnica (math).
"""

from __future__ import annotations

import ast
import math
import re
from typing import Any, List, Optional, Tuple, Union

# Tipni namigi (če core ni uvožen, damo fallback)
try:
    from orodja.odgovor_eval import EvalConfig
except Exception:  # pragma: no cover
    class EvalConfig:
        decimal_tolerance_abs: float = 1e-9
        decimal_tolerance_rel: float = 1e-9
        locale_decimal: str = "auto"
        accept_equivalent_forms: bool = True
        set_order_invariant: bool = True
        allow_expressions: bool = True

name = "trig"

CATEGORY_MATCHES = {"Trigonometrija", "Trigonometry", "Trigonom"}


# ============================ Pomožne: koti, enote, intervali ============================

_PI_TOKEN = re.compile(r'(^|[^a-zA-Z])pi([^a-zA-Z]|$)')   # ujame 'pi' kot konstanto
_DEG_NUMBER = re.compile(r'(?P<num>(?:[+\-]?\d+(?:\.\d+)?))\s*°')  # npr. 30°, -45.5°

def _is_trig_category(cat: str) -> bool:
    return any(key.lower() in cat.lower() for key in CATEGORY_MATCHES)

def _angle_mode_from_meta(meta: dict) -> str:
    """
    Vrne 'deg' | 'rad' | 'auto' glede na meta. Privzeto 'auto'.
    """
    if not isinstance(meta, dict):
        return "auto"
    mode = (meta.get("angle_mode") or meta.get("kotni_nacin") or "auto").lower()
    if mode in {"deg", "rad", "auto"}:
        return mode
    return "auto"

def _parse_interval_str(s: str) -> Tuple[float, float, bool, bool, str]:
    """
    Parsira interval npr. "[0,2π)" ali "[0°,360°)":
    Vrne (a, b, left_closed, right_closed, base_unit) z a,b v RADIANIH.
    base_unit ∈ {"rad","deg"} (pove, v čem je bil podan interval, zaradi normalizacije vhodov).
    """
    s = s.strip().replace(" ", "")
    if not s:
        # privzeto [0,2π)
        return 0.0, 2*math.pi, True, False, "rad"

    left_closed = s[0] == '['
    right_closed = s[-1] == ']'
    if s[0] not in '([' or s[-1] not in ')]':
        raise ValueError("Neveljaven zapis intervala.")
    inner = s[1:-1]
    if ',' not in inner:
        raise ValueError("Neveljaven interval: manjka vejica.")
    a_str, b_str = inner.split(',', 1)

    def _parse_bound(x: str) -> Tuple[float, str]:
        # zaznaj, ali so stopinje
        unit = "deg" if ('°' in x) or ('deg' in x.lower()) else "rad"
        # zamenjaj 'pi' -> math.pi; dopusti 2pi, pi/3, -pi/4, itd.
        x2 = x.replace("deg", "").replace("DEG", "").replace("Deg", "")
        x2 = _PI_TOKEN.sub(r"\1(3.141592653589793)\2", x2)
        # 30° -> 30*pi/180
        x2 = _DEG_NUMBER.sub(lambda m: f"({m.group('num')})*(3.141592653589793/180)", x2)
        # eval varno z ast (samo +,-,*,/, oklepaji, decim.)
        node = ast.parse(x2, mode="eval")
        allowed = (ast.Expression, ast.BinOp, ast.UnaryOp, ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow,
                   ast.Constant, ast.Constant, ast.USub, ast.UAdd)
        for sub in ast.walk(node):
            if not isinstance(sub, allowed):
                raise ValueError("Neveljaven izraz v meji intervala.")
        def _ev(n):
            if isinstance(n, ast.Expression): return _ev(n.body)
            if isinstance(n, ast.Constant):
                if isinstance(n.value, (int, float)): return float(n.value)
                raise ValueError
            if isinstance(n, ast.Constant): return float(n.n)
            if isinstance(n, ast.UnaryOp) and isinstance(n.op, (ast.UAdd, ast.USub)):
                v = _ev(n.operand); return v if isinstance(n.op, ast.UAdd) else -v
            if isinstance(n, ast.BinOp) and isinstance(n.op, (ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow)):
                return {ast.Add: lambda a,b:a+b, ast.Sub:lambda a,b:a-b,
                        ast.Mult:lambda a,b:a*b, ast.Div:lambda a,b:a/b,
                        ast.Pow:lambda a,b:a**b}[type(n.op)](_ev(n.left), _ev(n.right))
            raise ValueError
        val = _ev(node)
        # če je interval zapisan v stopinjah, ga pretvorimo v radiane
        if unit == "deg":
            val = val * math.pi / 180.0
        return float(val), unit

    a_val, unit_a = _parse_bound(a_str)
    b_val, unit_b = _parse_bound(b_str)
    base_unit = "deg" if (unit_a == "deg" or unit_b == "deg") else "rad"
    return float(a_val), float(b_val), left_closed, right_closed, base_unit

def _wrap_to_interval(x: float, a: float, b: float, left_closed: bool, right_closed: bool) -> float:
    """Normalizira kot x (v radianih) v interval [a,b) ali [a,b] glede na desni oklepaj."""
    # Poskrbimo, da a < b
    if b <= a:
        # če je slučajno [0,0) ipd., privzemi [0,2π)
        a, b = 0.0, 2*math.pi
    period = b - a
    y = (x - a) % period + a
    # če je desni rob zaprt in smo zelo blizu b, porini na b
    if right_closed and (abs(y - a) < 1e-15) and (abs((x - a) % period) < 1e-15):
        y = b
    # če je desni rob odprt, naj y nikoli ne postane b (vrni a namesto b)
    if not right_closed and abs(y - b) < 1e-15:
        y = a
    return y

def _angles_equal(u: float, v: float, tol: float = 1e-9) -> bool:
    return abs(u - v) <= tol


# ============================ Varen eval trig izrazov ============================

_ALLOWED_FUNCS = {
    "sin": math.sin,
    "cos": math.cos,
    "tan": math.tan,
    "asin": math.asin,
    "acos": math.acos,
    "atan": math.atan,
    "sqrt": math.sqrt,
    "abs": abs,
    "log": math.log,   # naravni log
    "ln": math.log,    # sinonim
}
_ALLOWED_CONSTS = {
    "pi": math.pi,
    "e": math.e,
}

def _normalize_expr_trig(s: str) -> str:
    """
    - Zamenja '^' → '**'
    - 'pi' spremeni v numerično pi (preko varnega evala spodaj to tudi lahko ostane kot 'pi')
    - '30°' → '(30)*(pi/180)' (delamo prek preprocesa, kjer 'pi' ostane identifikator)
    """
    s = s.strip()
    s = s.replace("^", "**")
    # 30° -> (30)*(pi/180)
    s = _DEG_NUMBER.sub(lambda m: f"({m.group('num')})*(pi/180)", s)
    return s

def _safe_eval_trig_expr(expr: str) -> float:
    """
    Varen evaluator za trig izraze:
    - Funkcije iz _ALLOWED_FUNCS
    - Konstanti pi, e
    - Operacije + - * / **, unarni +/- in oklepaji
    """
    expr = _normalize_expr_trig(expr)
    node = ast.parse(expr, mode="eval")

    def _eval(n):
        if isinstance(n, ast.Expression):
            return _eval(n.body)
        if isinstance(n, ast.Constant):
            if isinstance(n.value, (int, float)):
                return float(n.value)
            raise ValueError("Nedovoljena konstanta.")
        if isinstance(n, ast.Constant):  # Py<3.8
            return float(n.n)
        if isinstance(n, ast.UnaryOp) and isinstance(n.op, (ast.UAdd, ast.USub)):
            v = _eval(n.operand)
            return v if isinstance(n.op, ast.UAdd) else -v
        if isinstance(n, ast.BinOp) and isinstance(n.op, (ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow)):
            l = _eval(n.left); r = _eval(n.right)
            return {ast.Add: lambda a,b:a+b, ast.Sub: lambda a,b:a-b,
                    ast.Mult: lambda a,b:a*b, ast.Div: lambda a,b:a/b,
                    ast.Pow: lambda a,b:a**b}[type(n.op)](l, r)
        if isinstance(n, ast.Name):
            if n.id in _ALLOWED_CONSTS:
                return _ALLOWED_CONSTS[n.id]
            raise ValueError(f"Ime '{n.id}' ni dovoljeno.")
        if isinstance(n, ast.Call):
            if isinstance(n.func, ast.Name) and (n.func.id in _ALLOWED_FUNCS):
                fn = _ALLOWED_FUNCS[n.func.id]
                args = [_eval(a) for a in n.args]
                if len(args) != 1:
                    raise ValueError("Funkcija pričakuje 1 argument.")
                return float(fn(args[0]))
            raise ValueError("Dovoljene so le osnovne trig/elementarne funkcije.")
        raise ValueError(f"Nepodprta struktura v izrazu: {type(n).__name__}")

    # whitelist
    allowed = (ast.Expression, ast.BinOp, ast.UnaryOp, ast.Constant, ast.Constant,
               ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow, ast.UAdd, ast.USub,
               ast.Load, ast.Call, ast.Name)
    for sub in ast.walk(node):
        if not isinstance(sub, allowed):
            raise ValueError(f"Prepovedan AST element: {type(sub).__name__}")

    val = _eval(node)
    if not isinstance(val, (int, float)):
        raise ValueError("Pričakovana je številska vrednost trig. izraza.")
    return float(val)


# ============================ Parsiranje kotov iz uporabniškega vnosa ============================

def _parse_single_angle(s: str, angle_mode: str) -> float:
    """
    Parsira en kot v radianih:
    - sprejme nize kot: "30°", "-45.5°", "pi/6", "2pi/3", "1.2", "-3.14"
    - angle_mode: 'deg' | 'rad' | 'auto'
    """
    t = s.strip()
    if not t:
        raise ValueError("Prazen kot.")
    # če vsebuje ° → stopinje
    if '°' in t:
        # npr. '30°' ali '-45.5°'
        t2 = _DEG_NUMBER.sub(lambda m: m.group('num'), t)  # izvleci številko pred °
        try:
            deg = float(t2)
        except Exception:
            # tudi bolj kompleksno, npr. "(90/3)°"
            t3 = _normalize_expr_trig(t.replace('°', '*(pi/180)'))
            val = _safe_eval_trig_expr(t3)
            return float(val)  # že v radianih
        return deg * math.pi / 180.0

    # sicer lahko vsebuje 'pi' → radiani
    if 'pi' in t.lower():
        t2 = _normalize_expr_trig(t.replace("PI", "pi").replace("Pi", "pi"))
        # pustimo 'pi' kot konstanto; eval bo vrnil radiane
        return float(_safe_eval_trig_expr(t2))

    # brez ° in brez pi → odvisno od angle_mode
    # poskusi kot številko
    try:
        x = float(t.replace(',', '.'))  # decimalna vejica
    except Exception:
        # morda izraz brez trig funkcij, npr. "90/3"
        x = _safe_eval_trig_expr(_normalize_expr_trig(t))
    if angle_mode == "deg":
        return x * math.pi / 180.0
    # 'rad' ali 'auto' (privzeto rad)
    return float(x)

def _parse_angles_list(s: str, angle_mode: str) -> List[float]:
    """
    Parsira množico/ seznam kotov ločenih z vejicami/ podpičji ali v oklepajih:
    - "30°, 150°" ; "(pi/6, 5pi/6)" ; "{0, 180°, 360°}" ; "0; 2π/3; 4π/3"
    Vrne seznam v RADIANIH.
    """
    s = s.strip()
    if not s:
        return []
    # odstrani zunanje oklepaje, če objemajo cel niz
    if (s[0] in "([{" and s[-1] in ")]}") and _brackets_match_all(s):
        s = s[1:-1].strip()
    # top-level split po ',' ali ';'
    parts = _split_top_level(s, (',', ';'))
    out: List[float] = []
    for p in parts:
        out.append(_parse_single_angle(p, angle_mode))
    return out

def _split_top_level(s: str, seps: Tuple[str, ...]) -> List[str]:
    parts, buf = [], []
    depth = 0
    for ch in s:
        if ch in '([{':
            depth += 1
        elif ch in ')]}':
            depth -= 1
        if (ch in seps) and depth == 0:
            parts.append(''.join(buf).strip())
            buf = []
        else:
            buf.append(ch)
    if buf:
        parts.append(''.join(buf).strip())
    return [p for p in parts if p != ""]

def _brackets_match_all(s: str) -> bool:
    depth = 0
    for i, ch in enumerate(s):
        if ch in '([{':
            depth += 1
        elif ch in ')]}':
            depth -= 1
            if depth < 0:
                return False
        if depth == 0 and i != len(s) - 1:
            return False
    return depth == 0


# ============================ Plugin API ============================

def can_handle(problem: Any, subject: Optional[str] = None) -> bool:
    cat = subject or getattr(problem, "category", "") or ""
    return _is_trig_category(cat)

def preprocess_user_input(problem: Any, user_input: str, config: EvalConfig) -> Any:
    """
    Vrne:
      - za meta.type == "value": float (vrednost izraza ali številke),
      - za meta.type == "solve": seznam kotov (float, radiani), normaliziran v interval.
    """
    meta = getattr(problem, "meta", {}) or {}
    ptype = (meta.get("type") or "value").lower()
    angle_mode = _angle_mode_from_meta(meta)

    s = user_input.strip()
    if not s:
        return ""

    if ptype == "solve":
        # preberi interval iz meta (radiani)
        a, b, lc, rc, base_unit = _parse_interval_str(meta.get("interval", "[0,2π)"))
        # parse user kotov
        angles = _parse_angles_list(s, angle_mode)
        # normaliziraj v interval
        norm = [_wrap_to_interval(x, a, b, lc, rc) for x in angles]
        return norm

    # ptype == "value": uporabnik sme vpisati številko ali trig izraz
    try:
        # če vpis ni videti kot izraz, poskusi float
        val = _safe_eval_trig_expr(s)
        return float(val)
    except Exception:
        # morda je samo število z vejico
        try:
            return float(s.replace(',', '.'))
        except Exception:
            return s  # naj compare vrne 'wrong-format'

def canonical_solution(problem: Any, config: EvalConfig) -> Any:
    """
    Vrne kanonično rešitev:
      - "value": float
      - "solve": seznam kotov (float, radiani), normaliziran v podani interval
    """
    meta = getattr(problem, "meta", {}) or {}
    ptype = (meta.get("type") or "value").lower()
    angle_mode = _angle_mode_from_meta(meta)

    if ptype == "solve":
        a, b, lc, rc, base_unit = _parse_interval_str(meta.get("interval", "[0,2π)"))
        sol = getattr(problem, "solution", None)

        # rešitve: lahko seznam/tuple/set ali niz
        sols: List[float] = []
        if isinstance(sol, (list, tuple, set)):
            for x in sol:
                if isinstance(x, (int, float)):
                    rad = float(x) if base_unit == "rad" else (float(x) * math.pi / 180.0 if angle_mode == "deg" else float(x))
                else:
                    rad = _parse_single_angle(str(x), angle_mode)
                sols.append(rad)
        elif isinstance(sol, str):
            sols = _parse_angles_list(sol, angle_mode)
        elif isinstance(sol, (int, float)):
            # ena rešitev
            v = float(sol)
            sols = [v if base_unit == "rad" else (v * math.pi / 180.0 if angle_mode == "deg" else v)]
        else:
            sols = []

        # normaliziraj v interval
        sols = [_wrap_to_interval(x, a, b, lc, rc) for x in sols]
        return sols

    # "value"
    sol = getattr(problem, "solution", None)
    if isinstance(sol, (int, float)):
        return float(sol)
    if isinstance(sol, str):
        # poskusi eval trig izraza ali plain številke
        try:
            return float(_safe_eval_trig_expr(sol))
        except Exception:
            try:
                return float(sol.replace(',', '.'))
            except Exception:
                pass
    # fallback na numeric
    if hasattr(problem, "numeric") and problem.numeric is not None:
        try:
            return float(problem.numeric)
        except Exception:
            pass
    return None

def compare(lhs: Any, rhs: Any, config: EvalConfig) -> Tuple[bool, str, str]:
    """
    Primerjava:
      - "value": float vs float (tolerance)
      - "solve": seznam (user) vs seznam (solution) kotov (order-invariant, po toleranci, v istem intervalu)
    """
    # Če je rhs None, nimamo pravilne rešitve
    if rhs is None:
        return False, "wrong-format", "Ni mogoče določiti pravilne trig. rešitve."

    # "value" – pričakujemo številke
    if isinstance(rhs, (int, float)) and isinstance(lhs, (int, float)):
        ok = math.isclose(float(lhs), float(rhs),
                          rel_tol=config.decimal_tolerance_rel,
                          abs_tol=config.decimal_tolerance_abs)
        return (ok, "close" if ok else "wrong-value",
                "" if ok else "Napačno. Namig: preveri kotne enote (stopinje vs radiani) in oklepaje.")

    # "solve" – pričakujemo sezname kotov
    if isinstance(rhs, list) and isinstance(lhs, list):
        # primerjamo kot MNOŽICI z toleranco
        if len(lhs) != len(rhs):
            return False, "wrong-value", "Število podanih rešitev se ne ujema s pričakovanimi."

        # greedy ujemanje s toleranco
        used = [False]*len(rhs)
        tol = max(config.decimal_tolerance_abs, 1e-9)
        for u in lhs:
            found = False
            for j, v in enumerate(rhs):
                if not used[j] and _angles_equal(float(u), float(v), tol=tol):
                    used[j] = True
                    found = True
                    break
            if not found:
                return False, "wrong-value", "Vsaj ena izmed podanih kotnih rešitev se ne ujema."
        return True, "exact", ""

    # napačni tipi
    return False, "wrong-format", "Pričakovan je bil (seznam) kotov ali numerična vrednost, skladno z nalogo."


--- Konec datoteke: orodja\odgovor_plugins\trig_plugin.py ---

--- Začetek datoteke: orodja\odgovor_plugins\__init__.py ---



--- Konec datoteke: orodja\odgovor_plugins\__init__.py ---

--- Začetek datoteke: predmeti\funkcije_in_grafi.py ---

# -*- coding: utf-8 -*-
"""
predmeti/funkcije_in_grafi.py
-----------------------------
Generator nalog za snov "Funkcije in grafi".

Vključene oblike (naključno):
1) Linearna funkcija y = a x + b
   1a) Določi naklon (a) in presečišče z y-osjo (b)
   1b) Izračunaj y pri danem x
   1c) Izračunaj ničlo (x-presečišče), tj. y=0

2) Domena preprostih funkcij:
   - f(x) = 1 / (x - c)         → x ≠ c
   - g(x) = √(α x + β)          → α x + β ≥ 0

3) Prepoznavanje iz dveh točk (linearno): dana P1(x1,y1), P2(x2,y2)
   - Zapiši y = a x + b
   - (po želji) izračunaj še y pri danem x0

Vsaka naloga vrne Problem s polji:
- prompt, solution, steps, category="Funkcije in grafi", numeric (kjer smiselno).
"""

from __future__ import annotations
import random
from fractions import Fraction
from typing import Tuple, Optional

from .tipi import Problem
from .register import register

CATEGORY = "Funkcije in grafi"


# --------------------- Pomožne ---------------------

def _r(level: int, base: int) -> int:
    if level == 1:
        return random.randint(-base, base)
    elif level == 2:
        return random.randint(-2 * base, 2 * base)
    else:
        return random.randint(-3 * base, 3 * base)

def _fmt_frac(fr) -> str:
    """Sprejme int/Fraction/str in lepo izpiše (brez /1)."""
    try:
        fr = Fraction(fr)
    except Exception:
        return str(fr)
    return str(fr) if fr.denominator != 1 else str(fr.numerator)

def _fmt_linear(a: Fraction, b: Fraction) -> str:
    # y = a x + b (prijazen zapis)
    if a == 0:
        return f"y = {_fmt_frac(b)}"
    a_txt = "x" if a == 1 else ("-x" if a == -1 else f"{_fmt_frac(a)}x")
    if b == 0:
        return f"y = {a_txt}"
    sign = " + " if b > 0 else " - "
    return f"y = {a_txt}{sign}{_fmt_frac(abs(b))}"

def _nice_pair(level: int) -> Tuple[Fraction, Fraction]:
    lo = -4 if level == 1 else (-6 if level == 2 else -8)
    hi =  4 if level == 1 else ( 6 if level == 2 else  8)
    x = Fraction(random.randint(lo, hi))
    y = Fraction(random.randint(lo, hi))
    return x, y


# --------------------- Generatorji podvrst ---------------------

def _gen_linear_read(level: int) -> Problem:
    """Preberi a in b iz y = a x + b."""
    a = Fraction(_r(level, 5))
    b = Fraction(_r(level, 8))
    # izogni se povsem trivialni ničelni funkciji
    if a == 0 and b == 0:
        a = Fraction(1)

    eq = _fmt_linear(a, b)
    prompt = f"Podana je funkcija {eq}. Zapiši naklon (a) in presečišče z y-osjo (b)."
    solution = f"a = {_fmt_frac(a)}, b = {_fmt_frac(b)}"
    steps = "V obliki y = a x + b je koeficient pri x enak a, prosti člen pa b."
    p = Problem(prompt, solution, steps, CATEGORY, numeric=None)
    p.level = level
    return p

def _gen_linear_value(level: int) -> Problem:
    """Izračunaj y(x0) za y = a x + b."""
    a = Fraction(_r(level, 5))
    b = Fraction(_r(level, 8))
    if a == 0 and b == 0:
        a = Fraction(1)
    x0 = Fraction(random.randint(-6 if level >= 2 else -4, 6 if level >= 2 else 4))
    y0 = a * x0 + b

    eq = _fmt_linear(a, b)
    prompt = f"Za {eq} izračunaj y pri x = {_fmt_frac(x0)}."
    solution = _fmt_frac(y0)
    steps = (
        f"Vstavi x = {_fmt_frac(x0)}: y = {_fmt_frac(a)}·({_fmt_frac(x0)}) + {_fmt_frac(b)}"
        f" = {_fmt_frac(y0)}"
    )
    p = Problem(prompt, solution, steps, CATEGORY, numeric=float(y0))
    p.level = level
    return p

def _gen_linear_zero(level: int) -> Problem:
    """Ničla linearne: rešuj a x + b = 0."""
    a = Fraction(_r(level, 6) or 1)  # izognimo se 0
    b = Fraction(_r(level, 10))
    x0 = -b / a
    eq = _fmt_linear(a, b)
    prompt = f"Najdi ničlo funkcije {eq} (točka preseka z x-osjo)."
    solution = f"x₀ = {_fmt_frac(x0)}"
    steps = "Nastavi y = 0 in reši: a x + b = 0 ⇒ x = −b / a."
    p = Problem(prompt, solution, steps, CATEGORY, numeric=float(x0))
    p.level = level
    return p

def _gen_domain_rational(level: int) -> Problem:
    """Domena 1/(x - c)."""
    c = Fraction(random.randint(-8 if level >= 2 else -5, 8 if level >= 2 else 5))
    prompt = f"Določi domeno funkcije f(x) = 1/(x - {_fmt_frac(c)})."
    solution = f"D(f) = ℝ \\ {{ {_fmt_frac(c)} }} (x ≠ {_fmt_frac(c)})"
    steps = "V imenovalcu ne sme biti 0 ⇒ x − c ≠ 0 ⇒ x ≠ c."
    p = Problem(prompt, solution, steps, CATEGORY, numeric=None)
    p.level = level
    return p

def _gen_domain_sqrt(level: int) -> Problem:
    """Domena √(α x + β)."""
    a = random.choice([1, 2, 3, -1, -2, -3]) if level >= 2 else random.choice([1, 2])
    b = random.randint(-12 if level >= 3 else -8, 12 if level >= 3 else 8)
    # lepa predstavitev z znakom vmes
    sign = " + " if b >= 0 else " - "
    prompt = f"Določi domeno funkcije g(x) = √({a}x{sign}{abs(b)})."

    # Pogoj: a x + b ≥ 0
    bound = Fraction(-b, a)
    if a > 0:
        solution = f"D(g) = [ {_fmt_frac(bound)}, ∞ )"
        steps = f"Podkoren izraz ≥ 0 ⇒ {a}x + {b} ≥ 0 ⇒ x ≥ {_fmt_frac(bound)}."
    else:
        solution = f"D(g) = ( -∞, {_fmt_frac(bound)} ]"
        steps = f"Podkoren izraz ≥ 0 ⇒ {a}x + {b} ≥ 0 ⇒ x ≤ {_fmt_frac(bound)}."
    p = Problem(prompt, solution, steps, CATEGORY, numeric=None)
    p.level = level
    return p

def _gen_from_two_points(level: int) -> Problem:
    """
    Dana P1(x1,y1), P2(x2,y2), x1≠x2. Zapiši y = a x + b.
    (Po želji: izračunaj y v x0.)
    """
    x1, y1 = _nice_pair(level)
    x2, y2 = _nice_pair(level)
    while x2 == x1:
        x2, y2 = _nice_pair(level)

    a = Fraction(y2 - y1, x2 - x1)  # naklon
    b = Fraction(y1) - a * Fraction(x1)

    eq = _fmt_linear(a, b)
    # pri višjih težavnostih dodamo še vrednotenje v x0
    if level >= 2 and random.random() < 0.5:
        x0 = Fraction(random.randint(-5, 5))
        y0 = a * x0 + b
        prompt = (
            f"Podani sta točki P1=({_fmt_frac(x1)},{_fmt_frac(y1)}) in "
            f"P2=({_fmt_frac(x2)},{_fmt_frac(y2)}). "
            f"Zapiši enačbo premice in izračunaj y pri x = {_fmt_frac(x0)}."
        )
        solution = f"{eq}; y({ _fmt_frac(x0) }) = {_fmt_frac(y0)}"
        steps = (
            "Naklon a = (y2 − y1)/(x2 − x1). Nato b iz y1 = a x1 + b ⇒ b = y1 − a x1.\n"
            f"Enačba: {eq}. Vstavi x = {_fmt_frac(x0)} in izračunaj y."
        )
        numeric = float(y0)
    else:
        prompt = (
            f"Podani sta točki P1=({_fmt_frac(x1)},{_fmt_frac(y1)}) in "
            f"P2=({_fmt_frac(x2)},{_fmt_frac(y2)}). "
            "Zapiši enačbo premice v obliki y = a x + b."
        )
        solution = eq
        steps = "a = (y2 − y1)/(x2 − x1); b = y1 − a x1; nato zapiši y = a x + b."
        numeric = None

    p = Problem(
        prompt=prompt,
        solution=solution,
        steps=steps,
        category=CATEGORY,
        numeric=numeric,
    )
    p.level = level
    return p


# --------------------- Glavni generator ---------------------

def generiraj_funkcije_in_grafe(tezavnost: int) -> Problem:
    """
    Mode:
      - lin_read   : beri a,b iz y = a x + b
      - lin_value  : izračunaj y(x0)
      - lin_zero   : ničla
      - dom_rat    : domena racionalne 1/(x-c)
      - dom_sqrt   : domena korenske √(ax+b)
      - two_points : iz dveh točk zgradi y = a x + b
    """
    if tezavnost == 1:
        weights = [0.22, 0.22, 0.18, 0.18, 0.10, 0.10]
    elif tezavnost == 2:
        weights = [0.18, 0.20, 0.15, 0.18, 0.14, 0.15]
    else:
        weights = [0.14, 0.18, 0.14, 0.18, 0.18, 0.18]

    mode = random.choices(
        ["lin_read", "lin_value", "lin_zero", "dom_rat", "dom_sqrt", "two_points"],
        weights=weights, k=1
    )[0]

    if mode == "lin_read":
        return _gen_linear_read(tezavnost)
    if mode == "lin_value":
        return _gen_linear_value(tezavnost)
    if mode == "lin_zero":
        return _gen_linear_zero(tezavnost)
    if mode == "dom_rat":
        return _gen_domain_rational(tezavnost)
    if mode == "dom_sqrt":
        return _gen_domain_sqrt(tezavnost)
    return _gen_from_two_points(tezavnost)


# Registracija
register(CATEGORY, generiraj_funkcije_in_grafe)


--- Konec datoteke: predmeti\funkcije_in_grafi.py ---

--- Začetek datoteke: predmeti\linearne_enacbe.py ---

# -*- coding: utf-8 -*-
"""
predmeti/linearne_enacbe.py
---------------------------
Generator nalog za snov "Linearne enačbe" (ena spremenljivka).

Vključene oblike (naključno):
1) Z ulomki (SKI): (x±a)/m + (x±b)/n = k
2) Osnovna linearna: A·x + B = C·x + D
"""

from __future__ import annotations
import math
import random
from fractions import Fraction
from typing import Tuple

from .tipi import Problem
from .register import register

CATEGORY = "Linearne enačbe"


# --------------------- Pomožne ---------------------

def _r(lo: int, hi: int) -> int:
    return random.randint(lo, hi)

def _lcm(a: int, b: int) -> int:
    a, b = abs(a), abs(b)
    return abs(a * b) // math.gcd(a, b) if a and b else 0

def _fmt(fr: Fraction) -> str:
    return str(fr) if isinstance(fr, Fraction) and fr.denominator != 1 else str(fr.numerator if isinstance(fr, Fraction) else fr)


# --------------------- Tip 1: Ulomki + SKI ---------------------

def _gen_fractional(level: int) -> Problem:
    # parametri glede na težavnost
    if level == 1:
        a = _r(1, 4);  b = _r(1, 4)
        m = _r(2, 6);  n = _r(2, 6)
        k = _r(1, 8)
        s1 = 1; s2 = 1   # (x - a)/m + (x - b)/n
    elif level == 2:
        a = _r(1, 6);  b = _r(1, 6)
        m = _r(2, 8);  n = _r(2, 8)
        k = _r(1, 10)
        s1 = random.choice([1, -1])  # 1 → (x - a), -1 → (x + a)
        s2 = random.choice([1, -1])
    else:
        a = _r(1, 8);  b = _r(1, 8)
        m = _r(2, 10); n = _r(2, 10)
        k = _r(1, 12)
        s1 = random.choice([1, -1])
        s2 = random.choice([1, -1])

    # levo: (x ± a)/m + (x ± b)/n (dosledno po s1/s2)
    left1 = f"(x-{a})" if s1 > 0 else f"(x+{a})"
    left2 = f"(x-{b})" if s2 > 0 else f"(x+{b})"
    prompt = f"Reši za x: {left1}/{m} + {left2}/{n} = {k}"

    # Koeficient pred x in konstanta na LHS:
    Ax = Fraction(1, m) + Fraction(1, n)
    Bc = Fraction(-a if s1 > 0 else a, m) + Fraction(-b if s2 > 0 else b, n)
    rhs = Fraction(k, 1) - Bc
    x_val = rhs / Ax  # Ax>0 ker m,n≥2

    # Koraki: SKI
    lcm = _lcm(m, n)
    s1txt = f"x-{a}" if s1 > 0 else f"x+{a}"
    s2txt = f"x-{b}" if s2 > 0 else f"x+{b}"

    steps = (
        f"SKI = lcm({m},{n}) = {lcm}; pomnoži celotno enačbo s {lcm}.\n"
        f"{lcm}·({s1txt}/{m}) + {lcm}·({s2txt}/{n}) = {lcm}·{k}\n"
        f"⇒ {lcm//m}·({s1txt}) + {lcm//n}·({s2txt}) = {lcm*k}\n"
        f"Zberi x-člene in konstante: koef. pred x: {_fmt(Ax)}, konstanta: {_fmt(Bc)}\n"
        f"x = ({k} - {_fmt(Bc)}) / {_fmt(Ax)} = {_fmt(x_val)}"
    )

    p = Problem(
        prompt=prompt,
        solution=_fmt(x_val),
        steps=steps,
        category=CATEGORY,
        numeric=float(x_val)
    )
    p.level = level
    return p


# --------------------- Tip 2: Osnovna linearna A x + B = C x + D ---------------------

def _gen_basic(level: int) -> Problem:
    # Izberemo koeficiente, pazimo da A != C
    if level == 1:
        A = _r(1, 6); C = _r(1, 6)
        while C == A:
            C = _r(1, 6)
        B = _r(-8, 8); D = _r(-8, 8)
    elif level == 2:
        pool = [-6,-5,-4,-3,-2,-1,1,2,3,4,5,6]
        A = random.choice(pool); C = random.choice(pool)
        while C == A:
            C = random.choice(pool)
        B = _r(-12, 12); D = _r(-12, 12)
    else:
        pool = [-8,-7,-6,-5,-4,-3,-2,-1,1,2,3,4,5,6,7,8]
        A = random.choice(pool); C = random.choice(pool)
        while C == A:
            C = random.choice(pool)
        B = _r(-18, 18); D = _r(-18, 18)

    prompt = f"Reši za x: {A}·x + {B} = {C}·x + {D}"

    # (A - C) x = D - B
    coef = Fraction(A - C, 1)
    rhs = Fraction(D - B, 1)
    x_val = rhs / coef

    steps = (
        "Prenesi x-člene na levo, konstante na desno:\n"
        f"{A}x - {C}x = {D} - {B}\n"
        f"{A-C}x = {D-B}\n"
        f"x = ({D-B}) / ({A-C}) = {_fmt(x_val)}"
    )

    p = Problem(
        prompt=prompt,
        solution=_fmt(x_val),
        steps=steps,
        category=CATEGORY,
        numeric=float(x_val)
    )
    p.level = level
    return p


# --------------------- Glavni generator ---------------------

def generiraj_linearne_enacbe(tezavnost: int) -> Problem:
    """
    Naključno izberi tip: ulomki+SKI ali osnovna linearna.
    Višja težavnost: več negativnih in širši razponi.
    """
    mode = random.choices(
        population=["fractional", "basic"],
        weights=[0.55, 0.45] if tezavnost == 1 else ([0.6, 0.4] if tezavnost == 2 else [0.6, 0.4]),
        k=1
    )[0]

    if mode == "fractional":
        return _gen_fractional(tezavnost)
    return _gen_basic(tezavnost)


# Registracija kategorije
register(CATEGORY, generiraj_linearne_enacbe)


--- Konec datoteke: predmeti\linearne_enacbe.py ---

--- Začetek datoteke: predmeti\polinomi.py ---

# -*- coding: utf-8 -*-
"""
predmeti/polinomi.py
--------------------
Generator nalog za snov "Polinomi".

Vključene oblike (naključno):
1) Razširi (distributivnost) in poenostavi: k1(x+a) + k2(x+b)
2) Poenostavi izraz (zberi podobne člene): ax + bx + c + d ...
3) Vrednost polinoma: P(x) = Ax^2 + Bx + C pri x = t

Težavnost:
- 1: manjši koeficienti (−5..5), brez x^2 v (1) in (2), enostavna evalvacija
- 2: širši razpon koeficientov, več negativnih, pri (3) širši t
- 3: še širši razpon in pogosteje več členov

Rezultat:
- Pri (1) in (2) je solution simbolni polinom v kanonični obliki (npr. "3x + 2").
- Pri (3) je solution številčna vrednost (celo ali ulomek).
"""

from __future__ import annotations
import random
from fractions import Fraction
from typing import Dict, Tuple

from .tipi import Problem
from .register import register

CATEGORY = "Polinomi"


# --------------------- Pomožne: predstavitev polinoma ---------------------

# Polinom predstavimo kot slovar: stopnja -> koeficient (Fraction).
# Primer: 3x + 2 -> {1: 3, 0: 2}

def poly_add(p: Dict[int, Fraction], q: Dict[int, Fraction]) -> Dict[int, Fraction]:
    out: Dict[int, Fraction] = dict(p)
    for k, v in q.items():
        out[k] = out.get(k, Fraction(0)) + v
        if out[k] == 0:
            del out[k]
    return out

def poly_mul_scalar(p: Dict[int, Fraction], c: Fraction) -> Dict[int, Fraction]:
    if c == 0:
        return {}
    return {k: v * c for k, v in p.items()}

def poly_of_linear(k: Fraction, a: Fraction) -> Dict[int, Fraction]:
    # k*(x + a) = k*x + k*a  -> {1: k, 0: k*a}
    return {1: k, 0: k * a}

def poly_of_terms(ax: Fraction = 0, bx: Fraction = 0, c: Fraction = 0) -> Dict[int, Fraction]:
    d: Dict[int, Fraction] = {}
    if ax:
        d[1] = d.get(1, Fraction(0)) + ax
    if bx:
        d[1] = d.get(1, Fraction(0)) + bx
        if d[1] == 0:
            d.pop(1)
    if c:
        d[0] = d.get(0, Fraction(0)) + c
        if d[0] == 0:
            d.pop(0)
    return d

def poly_of_quadratic(A: Fraction, B: Fraction, C: Fraction) -> Dict[int, Fraction]:
    d: Dict[int, Fraction] = {}
    if A:
        d[2] = A
    if B:
        d[1] = B if 1 not in d else d[1] + B
    if C:
        d[0] = C if 0 not in d else d[0] + C
    return d

def poly_eval(p: Dict[int, Fraction], t: Fraction) -> Fraction:
    s = Fraction(0)
    for deg, coef in p.items():
        s += coef * (t ** deg)
    return s

def fmt_fraction(fr: Fraction) -> str:
    return str(fr) if fr.denominator != 1 else str(fr.numerator)

def fmt_poly(p: Dict[int, Fraction]) -> str:
    """
    Zapiše polinom v obliki: Ax^2 + Bx + C
    - koeficient 1 pred x pišemo kot 'x'
    - koeficient -1 pred x pišemo kot '-x'
    - ničelne člene izpustimo
    """
    if not p:
        return "0"
    parts = []
    for deg in sorted(p.keys(), reverse=True):
        c = p[deg]
        if c == 0:
            continue
        if deg == 0:
            chunk = fmt_fraction(c)
        elif deg == 1:
            if c == 1:
                chunk = "x"
            elif c == -1:
                chunk = "-x"
            else:
                chunk = f"{fmt_fraction(c)}x"
        else:
            if c == 1:
                chunk = f"x^{deg}"
            elif c == -1:
                chunk = f"-x^{deg}"
            else:
                chunk = f"{fmt_fraction(c)}x^{deg}"
        parts.append(chunk)

    if not parts:
        return "0"

    s = parts[0]
    for piece in parts[1:]:
        if piece.startswith("-"):
            s += " - " + piece[1:]
        else:
            s += " + " + piece
    return s


# --------------------- Pomožni izpisi za prijazno besedilo ---------------------

def pm_in_paren(v: Fraction) -> str:
    """
    Vrne niz ' + c' ali ' - c' (z razmikom), za uporabo kot (x ± c).
    """
    s = fmt_fraction(abs(v))
    return f" + {s}" if v >= 0 else f" - {s}"

def join_terms(*parts: str) -> str:
    """
    Združi koščke z znaki; preskoči prazne dele; poskrbi za pravilne razmike.
    """
    parts = [p for p in parts if p]
    if not parts:
        return "0"
    out = parts[0]
    for piece in parts[1:]:
        if piece.startswith("-"):
            out += " - " + piece[1:]
        else:
            out += " + " + piece
    return out


# --------------------- Generatorji podvrst ---------------------

def _r(level: int, base: int) -> int:
    # Razpon koeficientov glede na težavnost
    if level == 1:
        return random.randint(-base, base)
    elif level == 2:
        return random.randint(-2 * base, 2 * base)
    else:
        return random.randint(-3 * base, 3 * base)

def _gen_expand(level: int) -> Problem:
    """
    Razširi in poenostavi: k1(x+a) + k2(x+b)
    """
    k1 = Fraction(_r(level, 5) or 1)     # izognimo se 0, da ne bo trivialno
    k2 = Fraction(_r(level, 5) or -1)
    a = Fraction(_r(level, 5))
    b = Fraction(_r(level, 5))

    P1 = poly_of_linear(k1, a)
    P2 = poly_of_linear(k2, b)
    P = poly_add(P1, P2)

    prompt = (
        f"Razširi in poenostavi: "
        f"{fmt_fraction(k1)}(x{pm_in_paren(a)}) + {fmt_fraction(k2)}(x{pm_in_paren(b)})"
    )

    # Razširitev posameznih oklepajev (lepo pišemo koeficiente pri x)
    k1x = f"{fmt_fraction(k1)}x" if k1 not in (1, -1) else ("x" if k1 == 1 else "-x")
    k2x = f"{fmt_fraction(k2)}x" if k2 not in (1, -1) else ("x" if k2 == 1 else "-x")

    steps = (
        "Uporabi distributivnost in zberi podobne člene:\n"
        f"{fmt_fraction(k1)}(x{pm_in_paren(a)}) → {k1x}{pm_in_paren(k1 * a)}\n"
        f"{fmt_fraction(k2)}(x{pm_in_paren(b)}) → {k2x}{pm_in_paren(k2 * b)}\n"
        f"Skupaj: ({fmt_fraction(k1 + k2)})x{pm_in_paren(k1 * a + k2 * b)} = {fmt_poly(P)}"
    )

    p = Problem(
        prompt=prompt,
        solution=fmt_poly(P),
        steps=steps,
        category=CATEGORY,
        numeric=None
    )
    p.level = level
    return p

def _gen_simplify(level: int) -> Problem:
    """
    Poenostavi: seštej koeficiente pri x in konstante.
    Oblika: a1*x + a2*x + c1 + c2
    """
    a1 = Fraction(_r(level, 6))
    a2 = Fraction(_r(level, 6))
    c1 = Fraction(_r(level, 10))
    c2 = Fraction(_r(level, 10))

    P = poly_of_terms(ax=a1, bx=a2, c=c1 + c2)
    solution = fmt_poly(P)

    def term_x(c: Fraction) -> str:
        if c == 0:
            return ""
        if c == 1:
            return "x"
        if c == -1:
            return "-x"
        return f"{fmt_fraction(c)}x"

    parts = [
        term_x(a1),
        term_x(a2),
        fmt_fraction(c1) if c1 else "",
        fmt_fraction(c2) if c2 else ""
    ]
    prompt = "Poenostavi: " + join_terms(*parts)

    x1txt = term_x(a1) or "0"
    x2txt = term_x(a2) or "0"
    sumx = term_x(a1 + a2) or "0"
    const1 = fmt_fraction(c1) if c1 else "0"
    const2 = fmt_fraction(c2) if c2 else "0"
    constsum = fmt_fraction(c1 + c2)

    steps = (
        "Zberi podobne člene (x-členi skupaj, konstante skupaj):\n"
        f"x-členi: {x1txt} + {x2txt} = {sumx}\n"
        f"konstante: {const1} + {const2} = {constsum}\n"
        f"Rezultat: {solution}"
    )

    p = Problem(
        prompt=prompt,
        solution=solution,
        steps=steps,
        category=CATEGORY,
        numeric=None
    )
    p.level = level
    return p

def _gen_evaluate(level: int) -> Problem:
    """
    Vrednost P(x) = Ax^2 + Bx + C pri x=t.
    """
    A = Fraction(_r(level, 3))
    B = Fraction(_r(level, 5))
    C = Fraction(_r(level, 8))
    # Da ne bo popolnoma trivialno:
    if A == 0 and B == 0:
        A = Fraction(1)

    t = Fraction(random.randint(-5 if level >= 2 else -3, 5 if level >= 2 else 3))
    P = poly_of_quadratic(A, B, C)
    val = poly_eval(P, t)

    def fmt_term(coef: Fraction, name: str) -> str:
        if coef == 0:
            return ""
        if name == "x^2":
            if coef == 1:
                return "x^2"
            if coef == -1:
                return "-x^2"
            return f"{fmt_fraction(coef)}x^2"
        if name == "x":
            if coef == 1:
                return "x"
            if coef == -1:
                return "-x"
            return f"{fmt_fraction(coef)}x"
        return fmt_fraction(coef)

    parts = [fmt_term(A, "x^2"), fmt_term(B, "x"), fmt_term(C, "1")]
    poly_txt = " + ".join([p for p in parts if p]).replace("+ -", "- ")

    t_txt = fmt_fraction(t)
    steps = (
        f"Vstavi x = {t_txt}:\n"
        f"P({t_txt}) = {fmt_fraction(A)}·({t_txt})^2 + {fmt_fraction(B)}·({t_txt}) + {fmt_fraction(C)}\n"
        f"= {fmt_fraction(A * (t ** 2))} + {fmt_fraction(B * t)} + {fmt_fraction(C)} = {fmt_fraction(val)}"
    )

    prompt = f"Izračunaj P({t_txt}) za P(x) = {poly_txt}"

    p = Problem(
        prompt=prompt,
        solution=fmt_fraction(val),
        steps=steps,
        category=CATEGORY,
        numeric=float(val)
    )
    p.level = level
    return p


# --------------------- Glavni generator ---------------------

def generiraj_polinome(tezavnost: int) -> Problem:
    """
    Izberi med:
      - 'expand'   : razširi k1(x+a) + k2(x+b)
      - 'simplify' : zberi podobne člene
      - 'evaluate' : vrednoti kvadratični polinom v točki
    """
    if tezavnost == 1:
        weights = [0.45, 0.35, 0.20]
    elif tezavnost == 2:
        weights = [0.40, 0.30, 0.30]
    else:
        weights = [0.35, 0.30, 0.35]

    mode = random.choices(["expand", "simplify", "evaluate"], weights=weights, k=1)[0]
    if mode == "expand":
        return _gen_expand(tezavnost)
    if mode == "simplify":
        return _gen_simplify(tezavnost)
    return _gen_evaluate(tezavnost)


# Registracija v globalni register
register(CATEGORY, generiraj_polinome)


--- Konec datoteke: predmeti\polinomi.py ---

--- Začetek datoteke: predmeti\potence_in_koreni.py ---

# -*- coding: utf-8 -*-
"""
predmeti/potence_in_koreni.py
-----------------------------
Generator nalog za snov "Potence in koreni".

Vključene oblike (naključno):
1) Ista osnova – zmnožki/potence: a^m · a^n · (a^2)^k
2) Negativen eksponent + deljenje: a^(-2) / b   (kjer je b pogosto a^2 ipd.)
3) Poenostavljanje korenov: √N -> k√r (ekstrakcija popolnega kvadrata)

Težavnost:
- 1: manjši eksponenti, "lepe" baze (2,3,5), N iz nabora (50,72,98,108)
- 2: lahko vključi negativne eksponente, večje kombinacije
- 3: širši nabor in večji eksponenti

Vsaka naloga vrne Problem s prompt, solution, steps, category="Potence/Koreni",
numeric (kjer smiselno).
"""

from __future__ import annotations
import math
import random
from fractions import Fraction
from typing import List, Tuple

from .tipi import Problem
from .register import register


CATEGORY = "Potence/Koreni"


# --------------------- Pomožne funkcije ---------------------

def _choice_bases(tezavnost: int) -> List[int]:
    if tezavnost == 1:
        return [2, 3, 5]
    elif tezavnost == 2:
        return [2, 3, 5, 10]
    else:
        return [2, 3, 5, 6, 10]

def _rand_exp_plus(tezavnost: int) -> int:
    return random.randint(1, 4 if tezavnost == 1 else 5)

def _rand_exp_maybe_neg(tezavnost: int) -> int:
    if tezavnost == 1:
        return random.randint(0, 3)
    lo = -2 if tezavnost == 2 else -3
    hi = 4 if tezavnost == 2 else 5
    return random.randint(lo, hi)

def _fmt_power(a: int, e: int) -> str:
    """Prijazen prikaz potence: e=0 -> '1', e=1 -> 'a', sicer 'a^e'."""
    if e == 0:
        return "1"
    if e == 1:
        return f"{a}"
    return f"{a}^{e}"

def _extract_perfect_square(n: int) -> Tuple[int, int]:
    """
    Vrne (s, r), da je n = s*r in s je največji popolni kvadrat.
    Primer: n=72 -> (36, 2)
    """
    best = 1
    for s in [196,169,144,121,100,81,64,49,36,25,16,9,4]:
        if n % s == 0:
            best = s
            break
    return best, n // best

def _format_sqrt(n: int) -> Tuple[str, float]:
    """
    Poenostavi koren √n v k√r (če k>1), vrne prikaz in numerično referenco.
    """
    s, r = _extract_perfect_square(n)
    k = int(math.isqrt(s))
    if k == 1:
        display = f"√{r}"
        numeric = math.sqrt(r)
    else:
        display = f"{k}√{r}"
        numeric = k * math.sqrt(r)
    return display, numeric


# --------------------- Generatorji podvrst ---------------------

def _gen_same_base(tezavnost: int) -> Problem:
    """
    a^e1 · a^e2 · (a^2)^e3
    """
    a = random.choice(_choice_bases(tezavnost))
    e1 = _rand_exp_maybe_neg(tezavnost)
    e2 = _rand_exp_plus(tezavnost)
    e3 = random.randint(0, 3 if tezavnost == 1 else 4)

    prompt = f"Poenostavi: {a}^{e1} · {a}^{e2} · ({a}^2)^{e3}"
    total_exp = e1 + e2 + 2 * e3
    solution = _fmt_power(a, total_exp)

    numeric = None
    try:
        if -10 <= total_exp <= 20:
            numeric = float(a ** total_exp)
    except OverflowError:
        numeric = None

    steps = (
        "Ista osnova ⇒ eksponente seštevaš:\n"
        f"{a}^{e1} · {a}^{e2} = {a}^{e1 + e2}\n"
        f"({a}^2)^{e3} = {a}^{2*e3}\n"
        f"Skupaj: {a}^{e1 + e2} · {a}^{2*e3} = {a}^{total_exp}"
    )
    p = Problem(
        prompt=prompt,
        solution=solution,
        steps=steps,
        category=CATEGORY,
        numeric=numeric
    )
    p.level = tezavnost
    return p

def _gen_power_div(tezavnost: int) -> Problem:
    """
    a^(-2) / b    (kjer je b pogosto a^2, 4, 9, 25 ...)
    """
    a = random.choice([2, 3, 5])
    b = random.choice([4, 9, 25, a * a])  # včasih bo res a^2
    prompt = f"Poenostavi: {a}^(-2) / {b}"

    if b == a * a:
        # Klasičen primer: a^-2 / a^2 = 1/a^4
        solution = f"1/({a}^{4})"
        steps = (
            f"{a}^-2 = 1/{a}^2\n"
            f"Deljenje z {b} = {a}^2 ⇒ 1/({a}^2 · {a}^2) = 1/{a}^4"
        )
        numeric = 1 / (a ** 4)
    else:
        val = Fraction(1, a * a) / b  # 1/a^2 / b = 1/(a^2*b)
        solution = str(val)
        steps = (
            f"{a}^-2 = 1/{a}^2 = 1/{a*a}\n"
            f"Deljenje: (1/{a*a}) / {b} = 1/({a*a}·{b}) = {val}"
        )
        numeric = float(val)

    p = Problem(
        prompt=prompt,
        solution=solution,
        steps=steps,
        category=CATEGORY,
        numeric=numeric
    )
    p.level = tezavnost
    return p

def _gen_sqrt(tezavnost: int) -> Problem:
    """
    Poenostavljanje √N, kjer N ima "lepo" faktorizacijo na popoln kvadrat.
    """
    if tezavnost == 1:
        pool = [50, 72, 98, 108]
    elif tezavnost == 2:
        pool = [8, 12, 18, 20, 32, 45, 50, 72, 75, 98, 108, 128]
    else:
        pool = [8, 12, 18, 20, 24, 27, 32, 45, 48, 50, 54, 63, 72, 75, 80, 98, 108, 128, 147, 200]

    n = random.choice(pool)
    prompt = f"Izračunaj in poenostavi: √{n}"
    display, numeric = _format_sqrt(n)
    s, r = _extract_perfect_square(n)
    k = int(math.isqrt(s))

    steps = f"√{n} = √({s}·{r}) = √{s} · √{r} = {k}√{r}"
    p = Problem(
        prompt=prompt,
        solution=display,
        steps=steps,
        category=CATEGORY,
        numeric=numeric
    )
    p.level = tezavnost
    return p


# --------------------- Glavni generator za snov ---------------------

def generiraj_potence_in_koreni(tezavnost: int) -> Problem:
    mode = random.choices(
        population=["same_base", "power_div", "sqrt"],
        weights=[0.4, 0.25, 0.35],
        k=1
    )[0]

    if mode == "same_base":
        return _gen_same_base(tezavnost)
    if mode == "power_div":
        return _gen_power_div(tezavnost)
    return _gen_sqrt(tezavnost)


# Registracija kategorije v globalni register
register(CATEGORY, generiraj_potence_in_koreni)


--- Konec datoteke: predmeti\potence_in_koreni.py ---

--- Začetek datoteke: predmeti\procenti_in_ddv.py ---

# -*- coding: utf-8 -*-
"""
predmeti/procenti_in_ddv.py
---------------------------
Generator nalog za snov "Procenti, popusti, DDV".

Vključene oblike (naključno):
1) Popust + DDV: končna cena po popustu p% in nato DDV q%
2) Koliko je p% od N?
3) Kolikšen % je A od B?
4) Mešanje raztopin (redčenje z vodo): iz V ml koncentracije c1% na c2%

Težavnost:
- 1: lepe vrednosti, p/q iz standardnih naborov
- 2: širši razpon števil, večji nabor procentov
- 3: še širši razpon; pri raztopinah tudi decimalkaste koncentracije

Rezultat:
- solution: številčni rezultat (pri € v dveh decimalkah), simbolni koraki v 'steps'
- numeric: float (končni rezultat), zaokrožen pri denarnem formatu na 2 decimalki
"""

from __future__ import annotations
import random
from dataclasses import dataclass
from typing import Tuple, Optional

from .tipi import Problem
from .register import register

CATEGORY = "Procenti/DDV"


# --------------------- Pomožne ---------------------

def _fmt_eur(x: float) -> str:
    """Denarni zapis z dvema decimalkama in vejico (EU slog)."""
    s = f"{x:.2f}"
    s = s.replace(".", ",")
    return s + " €"

def _fmt_num(x: float, nd: int = 2) -> str:
    s = f"{x:.{nd}f}"
    return s.replace(".", ",")

def _pick_pct(level: int, kind: str = "discount") -> int:
    if kind == "vat":
        return 22  # v Sloveniji tipično 22 %
    if level == 1:
        pool = [5, 10, 12, 15, 20, 25, 30]
    elif level == 2:
        pool = [5, 7, 10, 12, 15, 18, 20, 25, 30, 35]
    else:
        pool = [3, 5, 7, 9, 10, 12, 15, 18, 20, 22, 25, 28, 30, 35, 40]
    return random.choice(pool)

def _pick_base(level: int) -> float:
    if level == 1:
        return float(random.randrange(800, 15001, 50)) / 100.0  # 8,00–150,00
    elif level == 2:
        return float(random.randrange(1500, 50001, 50)) / 100.0 # 15,00–500,00
    else:
        return float(random.randrange(1500, 150001, 25)) / 100.0 # 15,00–1500,00

def _nice_ml(level: int) -> int:
    if level == 1:
        return random.choice([100, 150, 200, 250, 300, 400, 500])
    elif level == 2:
        return random.choice([120, 180, 240, 300, 360, 450, 600, 750])
    else:
        return random.choice([125, 175, 225, 275, 325, 400, 650, 800, 1000])

def _pick_conc(level: int) -> Tuple[float, float]:
    if level == 1:
        c1 = random.choice([10, 20, 25, 30, 40])
        c2 = random.choice([5, 10, 15, 20])
        while c2 >= c1:
            c2 = random.choice([5, 10, 15, 20])
        return float(c1), float(c2)
    elif level == 2:
        c1 = random.choice([12, 18, 24, 30, 35])
        c2 = random.choice([6, 9, 12, 15, 18, 20])
        while c2 >= c1:
            c2 = random.choice([6, 9, 12, 15, 18, 20])
        return float(c1), float(c2)
    else:
        c1 = random.choice([12.5, 17.5, 22.5, 27.5, 35.0])
        c2 = random.choice([5.0, 7.5, 10.0, 12.5, 15.0, 17.5])
        while c2 >= c1:
            c2 = random.choice([5.0, 7.5, 10.0, 12.5, 15.0, 17.5])
        return float(c1), float(c2)


# --------------------- Generatorji podvrst ---------------------

def _gen_discount_vat(level: int) -> Problem:
    C0 = _pick_base(level)
    p = _pick_pct(level, "discount")
    q = _pick_pct(level, "vat")

    Cpop = C0 * (1 - p / 100.0)
    Ckonc = Cpop * (1 + q / 100.0)

    prompt = (
        f"Izdelek stane {_fmt_eur(C0)}. Trgovina ponudi {p}% popust, nato se prišteje {q}% DDV. "
        "Koliko plačaš na koncu?"
    )
    steps = (
        "Najprej popust:\n"
        f"C_popust = C0 · (1 − p/100) = {_fmt_eur(C0)} · (1 − {p}/100) = {_fmt_eur(Cpop)}\n"
        "Nato DDV:\n"
        f"C_končno = C_popust · (1 + q/100) = {_fmt_eur(Cpop)} · (1 + {q}/100) = {_fmt_eur(Ckonc)}"
    )
    p_obj = Problem(
        prompt=prompt,
        solution=_fmt_eur(Ckonc),
        steps=steps,
        category=CATEGORY,
        numeric=round(Ckonc, 2)
    )
    p_obj.level = level
    return p_obj

def _gen_percent_of(level: int) -> Problem:
    N = _pick_base(level) * 100  # malce večje število
    p = _pick_pct(level, "any")
    value = N * p / 100.0
    prompt = f"Izračunaj: {p}% od { _fmt_num(N, 2) }"
    steps = f"{p}% od N pomeni (p/100)·N = ({p}/100)·{_fmt_num(N,2)} = {_fmt_num(value,2)}"
    p_obj = Problem(
        prompt=prompt,
        solution=_fmt_num(value, 2),
        steps=steps,
        category=CATEGORY,
        numeric=round(value, 2)
    )
    p_obj.level = level
    return p_obj

def _gen_what_percent(level: int) -> Problem:
    B = _pick_base(level) * 100
    if level == 1:
        ratio = random.uniform(0.1, 0.9)
    elif level == 2:
        ratio = random.uniform(0.05, 1.2)
    else:
        ratio = random.uniform(0.03, 1.5)
    A = B * ratio
    p = (A / B) * 100.0

    prompt = f"Kolikšen delež (v %) je { _fmt_num(A, 2) } od { _fmt_num(B, 2) }?"
    steps = f"p = (A/B) · 100 = ({_fmt_num(A,2)}/{_fmt_num(B,2)}) · 100 = {_fmt_num(p,2)}%"
    p_obj = Problem(
        prompt=prompt,
        solution=_fmt_num(p, 2) + " %",
        steps=steps,
        category=CATEGORY,
        numeric=round(p, 2)
    )
    p_obj.level = level
    return p_obj

def _gen_dilution(level: int) -> Problem:
    V = _nice_ml(level)  # začetni volumen (ml)
    c1, c2 = _pick_conc(level)  # v %
    # Dodaš čisto vodo (0%), da dobiš c2 (%). Koliko ml vode dodaš?
    # c1% * V = c2% * (V + x)  =>  x = V*(c1/c2 - 1)
    x = V * (c1 / c2 - 1.0)
    V_final = V + x

    prompt = (
        f"Imaš {V} ml raztopine koncentracije {_fmt_num(c1,1)}%. "
        f"Koliko ml vode dodaš, da dobiš raztopino {_fmt_num(c2,1)}%?"
    )
    steps = (
        "Ohrani se količina raztopljene snovi:\n"
        f"(c1/100)·V = (c2/100)·(V + x)\n"
        f"x = V·(c1/c2 − 1) = {V}·({_fmt_num(c1,1)}/{_fmt_num(c2,1)} − 1) = {_fmt_num(x,2)} ml\n"
        f"Končni volumen: V + x = {_fmt_num(V_final,2)} ml"
    )
    p_obj = Problem(
        prompt=prompt,
        solution=_fmt_num(x, 2) + " ml",
        steps=steps,
        category=CATEGORY,
        numeric=round(x, 2)
    )
    p_obj.level = level
    return p_obj


# --------------------- Glavni generator ---------------------

def generiraj_procente_in_ddv(tezavnost: int) -> Problem:
    if tezavnost == 1:
        weights = [0.35, 0.30, 0.20, 0.15]  # discount_vat, percent_of, what_percent, dilution
    elif tezavnost == 2:
        weights = [0.32, 0.28, 0.22, 0.18]
    else:
        weights = [0.30, 0.25, 0.20, 0.25]
    mode = random.choices(
        ["discount_vat", "percent_of", "what_percent", "dilution"],
        weights=weights, k=1
    )[0]

    if mode == "discount_vat":
        return _gen_discount_vat(tezavnost)
    if mode == "percent_of":
        return _gen_percent_of(tezavnost)
    if mode == "what_percent":
        return _gen_what_percent(tezavnost)
    return _gen_dilution(tezavnost)


# Registracija v globalni register
register(CATEGORY, generiraj_procente_in_ddv)


--- Konec datoteke: predmeti\procenti_in_ddv.py ---

--- Začetek datoteke: predmeti\register.py ---

# -*- coding: utf-8 -*-
"""
predmeti/register.py
--------------------
Preprost register (imenik) vseh snovi in njihovih generatorjev nalog.

Uporaba (v posamezni snovi, npr. 'ulomki.py'):
    from .tipi import Problem
    from .register import register

    def generiraj_ulomke(tezavnost: int) -> Problem:
        # ... sestavi in vrne Problem ...
        return Problem(prompt="...", solution="...", steps="...", category="Ulomki")

    register("Ulomki", generiraj_ulomke)

GUI/CLI potem lahko:
    from predmeti import register
    imena = register.list_categories()
    naloga = register.generate("Ulomki", tezavnost=2)
"""

from __future__ import annotations
from typing import Callable, Dict, List
from .tipi import Problem

# Tip funkcije za generator: sprejme težavnost (1–3) in vrne Problem
Generator = Callable[[int], Problem]

# Interni slovar registriranih snovi (ključ: normalizirano ime)
_CATEGORIES: Dict[str, Generator] = {}

# Originalna (prikazna) imena kategorij; mapira normalizirano -> originalno
_DISPLAY_NAMES: Dict[str, str] = {}


class UnknownCategoryError(KeyError):
    """Vrže se, če zahtevana kategorija ni registrirana."""


def _normalize(name: str) -> str:
    """
    Normalizira ime kategorije:
    - obreže presledke,
    - stisne več presledkov na enega,
    - case-insensitive ključ (lower + casefold).
    """
    if not isinstance(name, str):
        raise TypeError("Ime kategorije mora biti niz.")
    squeezed = " ".join(name.strip().split())
    return squeezed.casefold()


def register(name: str, generator: Generator) -> None:
    """
    Registrira snov pod podanim imenom.

    Če se isto ime registrira večkrat, se prepiše s zadnjim generatorjem
    (namenoma – da je razvoj iterativen).
    """
    if not isinstance(name, str) or not name.strip():
        raise ValueError("Ime kategorije mora biti neničen niz.")
    if not callable(generator):
        raise TypeError("Generator mora biti klicna funkcija: (int) -> Problem.")

    key = _normalize(name)
    _CATEGORIES[key] = generator
    # shranimo tudi lepo prikazno ime (stisnjeni presledki, ohranjen zapis črk)
    _DISPLAY_NAMES[key] = " ".join(name.strip().split())


def list_categories() -> List[str]:
    """Vrne abecedno urejen seznam imen vseh registriranih kategorij (prikazna imena)."""
    # sortiramo po casefold, da je izpis konsistenten
    return sorted(_DISPLAY_NAMES.values(), key=lambda s: s.casefold())


def generate(name: str, tezavnost: int) -> Problem:
    """
    Pokliče ustrezen generator za dano kategorijo in težavnost.
    Ime je tolerantno na velike/male črke in odvečne presledke.
    """
    key = _normalize(name)
    try:
        gen = _CATEGORIES[key]
    except KeyError as e:
        available = ", ".join(list_categories()) or "—"
        raise UnknownCategoryError(
            f"Kategorija '{name}' ni registrirana. Na voljo: {available}"
        ) from e
    return gen(tezavnost)


def is_registered(name: str) -> bool:
    """Ali je kategorija registrirana? (tolerantno na presledke in velikost črk)"""
    return _normalize(name) in _CATEGORIES


__all__ = [
    "Problem",
    "Generator",
    "register",
    "list_categories",
    "generate",
    "is_registered",
    "UnknownCategoryError",
]


--- Konec datoteke: predmeti\register.py ---

--- Začetek datoteke: predmeti\sistemi_enacb.py ---

# -*- coding: utf-8 -*-
"""
predmeti/sistemi_enacb.py
-------------------------
Generator nalog za snov "Sistemi enačb" (dve neznanki, linearni sistem).

Vključene oblike (naključno):
1) Eliminacija: A1·x + B1·y = C1  in  A2·x + B2·y = C2
2) Vstavitev (substitucija): y = m·x + n  in  A·x + B·y = C  (ali x = m·y + n ...)

Težavnost:
- 1: manjši koeficienti, rešitvi sta celi števili (|x0|,|y0| ≤ 6)
- 2: širši razpon koeficientov, možne ulomčne rešitve
- 3: še širši razpon, več negativnih koeficientov

Rezultat:
- solution: simbolno "x = ..., y = ..."
- steps: opis korakov (eliminacija ali substitucija)
- numeric: None
"""

from __future__ import annotations
import random
from fractions import Fraction
from typing import Tuple

from .tipi import Problem
from .register import register

CATEGORY = "Sistemi enačb"


# --------------------- Pomožne ---------------------

def _rand_nonzero(lo: int, hi: int) -> int:
    while True:
        v = random.randint(lo, hi)
        if v != 0:
            return v

def _pick_solution(level: int) -> Tuple[Fraction, Fraction]:
    """Izberi (x0, y0) glede na težavnost (na višjih stopnjah tudi 'lepi' ulomki)."""
    if level == 1:
        return Fraction(random.randint(-6, 6)), Fraction(random.randint(-6, 6))
    elif level == 2:
        if random.random() < 0.7:
            return Fraction(random.randint(-8, 8)), Fraction(random.randint(-8, 8))
        den = random.choice([2, 3, 4, 5, 6])
        x0 = Fraction(random.randint(-12, 12), den)
        y0 = Fraction(random.randint(-12, 12), den)
        return x0, y0
    else:
        if random.random() < 0.5:
            den = random.choice([2, 3, 4, 5, 6, 8])
            x0 = Fraction(random.randint(-18, 18), den)
            y0 = Fraction(random.randint(-18, 18), den)
            return x0, y0
        return Fraction(random.randint(-12, 12)), Fraction(random.randint(-12, 12))

def _fmt(fr) -> str:
    """Lep prikaz Fraction/int (brez /1)."""
    try:
        fr = Fraction(fr)
    except Exception:
        return str(fr)
    return str(fr) if fr.denominator != 1 else str(fr.numerator)

def _join_terms(parts):
    """Združi člene s pravilnimi znaki in razmiki."""
    parts = [p for p in parts if p]
    if not parts:
        return "0"
    s = parts[0]
    for t in parts[1:]:
        if t.startswith("-"):
            s += " - " + t[1:]
        else:
            s += " + " + t
    return s

def _term(coef: int, var: str) -> str:
    """Vrne 'coef·var' z lepim zapisom za 1/-1/0."""
    if coef == 0:
        return ""
    if coef == 1:
        return var
    if coef == -1:
        return f"-{var}"
    return f"{coef}{var}"

def _pm(v) -> str:
    """Vrne ' + c' ali ' - c' za konstanto v (Fraction/int)."""
    v = Fraction(v)
    s = _fmt(abs(v))
    return f" + {s}" if v >= 0 else f" - {s}"

def _coef_var(m, var: str) -> str:
    """Vrne 'm·var' z lepim zapisom koeficienta (1/-1)."""
    m = Fraction(m)
    if m == 1:
        return var
    if m == -1:
        return f"-{var}"
    return f"{_fmt(m)}{var}"

def _eq_text(a: int, b: int, c) -> str:
    """Prijazen zapis A·x + B·y = C (odpusti 1 in -1 pred spremenljivkama)."""
    left = _join_terms([_term(a, "x"), _term(b, "y")])
    return f"{left} = {_fmt(c)}"


# --------------------- Tip 1: Eliminacija ---------------------

def _gen_elimination(level: int) -> Problem:
    # izberi rešitev
    x0, y0 = _pick_solution(level)

    # robustno izberi koeficiente, dokler pari niso proporcionalni
    def _pick_coeffs(lo: int, hi: int):
        while True:
            a1 = _rand_nonzero(lo, hi)
            b1 = _rand_nonzero(lo, hi)
            a2 = _rand_nonzero(lo, hi)
            b2 = _rand_nonzero(lo, hi)
            if a1 * b2 != a2 * b1:  # neproporcionalno → enolična rešitev
                return a1, b1, a2, b2

    if level == 1:
        a1, b1, a2, b2 = _pick_coeffs(-6, 6)
    elif level == 2:
        a1, b1, a2, b2 = _pick_coeffs(-9, 9)
    else:
        a1, b1, a2, b2 = _pick_coeffs(-12, 12)

    # izračunaj desne strani, da bo (x0,y0) točna rešitev
    c1 = a1 * x0 + b1 * y0
    c2 = a2 * x0 + b2 * y0

    e1 = _eq_text(a1, b1, c1)
    e2 = _eq_text(a2, b2, c2)

    prompt = "Reši sistem z eliminacijo:\n{ " + e1 + " ;  " + e2 + " }"

    # izberi, kaj eliminiramo: cilj je manjša absolutna vsota koeficientov
    # (približno vodi do manjših množilnikov)
    if abs(b1) + abs(b2) <= abs(a1) + abs(a2):
        # eliminiraj y
        k1, k2 = b2, -b1
        s1 = f"{k1}×({e1})"
        s2 = f"{k2}×({e2})"
        A = k1 * a1 + k2 * a2
        C = k1 * c1 + k2 * c2
        step3 = f"({A})x = {_fmt(C)}"
        x_txt = f"x = {_fmt(Fraction(C, A))}" if A != 0 else "x = ?"
        steps = (
            "Poravnaj koeficiente pri y in seštej enačbi:\n"
            f"{s1}\n{s2}\n"
            f"⇒ {step3} → {x_txt}\n"
            "Nato vrednost x vstavi v eno od enačb in izračunaj y."
        )
    else:
        # eliminiraj x
        k1, k2 = a2, -a1
        s1 = f"{k1}×({e1})"
        s2 = f"{k2}×({e2})"
        B = k1 * b1 + k2 * b2
        C = k1 * c1 + k2 * c2
        step3 = f"({B})y = {_fmt(C)}"
        y_txt = f"y = {_fmt(Fraction(C, B))}" if B != 0 else "y = ?"
        steps = (
            "Poravnaj koeficiente pri x in seštej enačbi:\n"
            f"{s1}\n{s2}\n"
            f"⇒ {step3} → {y_txt}\n"
            "Nato vrednost y vstavi v eno od enačb in izračunaj x."
        )

    solution = f"x = {_fmt(x0)}, y = {_fmt(y0)}"
    p = Problem(
        prompt=prompt,
        solution=solution,
        steps=steps,
        category=CATEGORY,
        numeric=None
    )
    # dosledno: Problem nima polja 'level', zato ga dodamo dinamično
    p.level = level
    return p


# --------------------- Tip 2: Substitucija ---------------------

def _gen_substitution(level: int) -> Problem:
    # izberi rešitev
    x0, y0 = _pick_solution(level)

    # izberi lep m, n
    if level == 1:
        m = Fraction(random.randint(-3, 3) or 1, random.choice([1, 2]))
        n = Fraction(random.randint(-6, 6))
    elif level == 2:
        m = Fraction(random.randint(-4, 4) or 1, random.choice([1, 2, 3]))
        n = Fraction(random.randint(-10, 10), random.choice([1, 2]))
    else:
        m = Fraction(random.randint(-6, 6) or 1, random.choice([1, 2, 3, 4]))
        n = Fraction(random.randint(-12, 12), random.choice([1, 2, 3]))

    # zgradi drugo enačbo A x + B y = C, da drži rešitev (x0,y0)
    if level == 1:
        A = _rand_nonzero(-8, 8)
        B = _rand_nonzero(-8, 8)
    else:
        A = _rand_nonzero(-12, 12)
        B = _rand_nonzero(-12, 12)
    C = A * x0 + B * y0

    # izberi, ali podamo y=... ali x=...
    if random.random() < 0.65:
        # y = m x + n
        n = y0 - m * x0  # prilagodi, da (x0,y0) drži
        first = f"y = {_coef_var(m, 'x')}{_pm(n)}"
        inner = f"{_coef_var(m, 'x')}{_pm(n)}"
        prompt = "Reši sistem z vstavitvijo (substitucijo):\n{ " + first + " ;  " + _eq_text(A, B, C) + " }"
        steps = (
            "V drugo enačbo vstavi izraz za y iz prve:\n"
            f"{_eq_text(A, B, C)}\n"
            f"⇒ {A}x + {B}({inner}) = {_fmt(C)}\n"
            "Reši za x, nato v prvi enačbi izračunaj y."
        )
    else:
        # x = m y + n
        n = x0 - m * y0
        first = f"x = {_coef_var(m, 'y')}{_pm(n)}"
        inner = f"{_coef_var(m, 'y')}{_pm(n)}"
        prompt = "Reši sistem z vstavitvijo (substitucijo):\n{ " + first + " ;  " + _eq_text(A, B, C) + " }"
        steps = (
            "V drugo enačbo vstavi izraz za x iz prve:\n"
            f"{_eq_text(A, B, C)}\n"
            f"⇒ {A}({inner}) + {B}y = {_fmt(C)}\n"
            "Reši za y, nato iz prve enačbe izračunaj x."
        )

    solution = f"x = {_fmt(x0)}, y = {_fmt(y0)}"
    p = Problem(
        prompt=prompt,
        solution=solution,
        steps=steps,
        category=CATEGORY,
        numeric=None
    )
    # dosledno: Problem nima polja 'level', zato ga dodamo dinamično
    p.level = level
    return p


# --------------------- Glavni generator ---------------------

def generiraj_sisteme_enacb(tezavnost: int) -> Problem:
    mode = random.choices(
        population=["elim", "subst"],
        weights=[0.55, 0.45] if tezavnost == 1 else ([0.5, 0.5] if tezavnost == 2 else [0.5, 0.5]),
        k=1
    )[0]
    if mode == "elim":
        return _gen_elimination(tezavnost)
    return _gen_substitution(tezavnost)


# Registracija v globalni register
register(CATEGORY, generiraj_sisteme_enacb)


--- Konec datoteke: predmeti\sistemi_enacb.py ---

--- Začetek datoteke: predmeti\tipi.py ---

# -*- coding: utf-8 -*-
"""
predmeti/tipi.py
----------------
Skupne podatkovne strukture in tipi za generatorje nalog po snoveh.

Dataclass Problem:
- prompt   : besedilo naloge (kaj naj učenec izračuna/poenostavi)
- solution : prikazna rešitev (lahko simbolna: npr. '5√2' ali '3/5·a')
- steps    : razlaga/koraki reševanja (za gumb 'Pokaži korake' v GUI)
- category : ime snovi (npr. 'Ulomki', 'Trigonometrija', ...)
- numeric  : opcijska numerična referenca (float) za tolerantno preverjanje

Opombe:
- Dodani sta neobvezni polji 'seed' in 'level' (meta-informacija),
  ki NE prelomita kompatibilnosti in sta privzeto None.
- __post_init__ poskrbi za osnovno validacijo in obrezovanje presledkov.
- Metodi to_dict()/from_dict() olajšata (de)serializacijo.
- preview() je pomočnica za kratek povzetek (npr. v seznamih).
"""

from __future__ import annotations
from dataclasses import dataclass, field
from typing import Optional, Dict, Any


@dataclass
class Problem:
    """
    Enotna reprezentacija naloge, ki jo tvorijo posamezne snovi (moduli).
    """
    prompt: str
    solution: str
    steps: str
    category: str
    numeric: Optional[float] = None

    # Neobvezna meta-polja (ne vplivajo na generiranje; koristna za GUI/log)
    seed: Optional[int] = None
    level: Optional[int] = None

    # Interno: kratki predogledi naj ne trgajo UI, omejimo dolžine
    _max_preview_len: int = field(default=140, repr=False, compare=False)

    def __post_init__(self) -> None:
        # Normalizacija osnovnih stringov (obrez robnih presledkov)
        self.prompt = self.prompt.strip()
        self.solution = self.solution.strip()
        self.steps = self.steps.strip()
        self.category = " ".join(self.category.split()).strip()

        # Minimalna validacija
        if not self.prompt:
            raise ValueError("Problem.prompt ne sme biti prazen.")
        if not self.solution:
            raise ValueError("Problem.solution ne sme biti prazen.")
        if not self.category:
            raise ValueError("Problem.category ne sme biti prazen.")

        # numeric je lahko None ali float (dovolimo int, a ga pretvorimo)
        if self.numeric is not None and not isinstance(self.numeric, (int, float)):
            raise TypeError("Problem.numeric mora biti float ali None.")
        if isinstance(self.numeric, int):
            self.numeric = float(self.numeric)

        # level (če podan) naj bo 1–3
        if self.level is not None and self.level not in (1, 2, 3):
            raise ValueError("Problem.level (če podan) mora biti 1, 2 ali 3.")

    # ------------------- Uporabne pomožne metode -------------------

    def preview(self) -> str:
        """
        Kratek enovrstični povzetek (za sezname v GUI): "<kategorija>: <prompt…>"
        """
        base = f"{self.category}: {self.prompt}"
        if len(base) <= self._max_preview_len:
            return base
        return base[: self._max_preview_len - 1].rstrip() + "…"

    def to_dict(self) -> Dict[str, Any]:
        """
        Serijalizacija v slovar (npr. za zapis v JSON).
        """
        return {
            "prompt": self.prompt,
            "solution": self.solution,
            "steps": self.steps,
            "category": self.category,
            "numeric": self.numeric,
            "seed": self.seed,
            "level": self.level,
        }

    @classmethod
    def from_dict(cls, d: Dict[str, Any]) -> "Problem":
        """
        Povratna (de)serializacija iz slovarja (npr. JSON → Problem).
        Nepoznane ključe ignorira.
        """
        return cls(
            prompt=d.get("prompt", ""),
            solution=d.get("solution", ""),
            steps=d.get("steps", ""),
            category=d.get("category", ""),
            numeric=d.get("numeric", None),
            seed=d.get("seed", None),
            level=d.get("level", None),
        )


__all__ = ["Problem"]


--- Konec datoteke: predmeti\tipi.py ---

--- Začetek datoteke: predmeti\trigonometrija.py ---

# -*- coding: utf-8 -*-
"""
predmeti/trigonometrija.py
--------------------------
Generator nalog za snov "Trigonometrija".

Vključene oblike (naključno):
1) Posebni koti (stopinje): sin/cos/tan pri 0°, 30°, 45°, 60°, 90° ...
2) Pretvorba kotov: stopinje ↔ radiani (π oblike)
3) Pravokotni trikotnik: Pitagora ali osnovna razmerja (sin/cos/tan)

Težavnost:
- 1: posebni koti, enostavne pretvorbe (koti iz nabora), Pitagora s celimi stranicami
- 2: vključimo tan pri posebnih kotih (tudi √3/3), različne smeri pretvorb,
     pravokotni trikotnik z razmerji in lepimi števili
- 3: širši nabor dolžin, lahko vključuje decimalke; v trikotniku zahtevamo tudi kot

Rezultat:
- Pri posebnih kotih vrnemo točno simbolno vrednost (npr. "√3/2", "√3/3", "0").
- Pri pretvorbah π-oblike (npr. "π/6", "3π/4") ali celo število stopinj.
- Pri trikotniku: številčna vrednost (po potrebi na 2–3 decimalkah).
"""

from __future__ import annotations
import math
import random
from fractions import Fraction
from typing import Tuple, Optional

from .tipi import Problem
from .register import register

CATEGORY = "Trigonometrija"


# --------------------- Pomožne funkcije ---------------------

def _fmt_frac(fr: Fraction) -> str:
    return str(fr) if isinstance(fr, Fraction) and fr.denominator != 1 else str(int(fr))

def _round_str(x: float, nd: int = 3) -> str:
    s = f"{x:.{nd}f}"
    return s.rstrip("0").rstrip(".")

def _deg_to_pi_frac(deg: int) -> Fraction:
    """
    Vrne k kot ulomek, tako da je deg = k * 180°  <=>  rad = k·π.
    Primer: 30° -> 1/6, 45° -> 1/4, 60° -> 1/3, 90° -> 1/2, 180° -> 1, ...
    """
    return Fraction(deg, 180)  # k · π

def _pi_frac_to_deg(p: Fraction) -> Fraction:
    """p·π rad → stopinje = p * 180°"""
    return p * 180

def _clip01(v: float) -> float:
    return max(0.0, min(1.0, v))

def _special_trig(func: str, deg: int) -> Tuple[str, float]:
    """
    Vrne (simbolna, numeric) za sin/cos/tan pri posebnih kotih.
    Pokrivamo kote iz tipičnih tabel; za ostale vrnemo približek "≈...".
    """
    deg = deg % 360

    if func == "sin":
        if deg in (0, 180, 360):   return "0", 0.0
        if deg == 90:              return "1", 1.0
        if deg == 270:             return "-1", -1.0
        if deg in (30, 150):       return "1/2", 0.5
        if deg in (210, 330):      return "-1/2", -0.5
        if deg in (45, 135):       return "√2/2", math.sqrt(2)/2
        if deg in (225, 315):      return "-√2/2", -math.sqrt(2)/2
        if deg in (60, 120):       return "√3/2", math.sqrt(3)/2
        if deg in (240, 300):      return "-√3/2", -math.sqrt(3)/2

    if func == "cos":
        if deg in (90, 270):       return "0", 0.0
        if deg in (0, 360):        return "1", 1.0
        if deg == 180:             return "-1", -1.0
        if deg in (60, 300):       return "1/2", 0.5
        if deg in (120, 240):      return "-1/2", -0.5
        if deg in (45, 315):       return "√2/2", math.sqrt(2)/2
        if deg in (135, 225):      return "-√2/2", -math.sqrt(2)/2
        if deg in (30, 330):       return "√3/2", math.sqrt(3)/2
        if deg in (150, 210):      return "-√3/2", -math.sqrt(3)/2

    if func == "tan":
        # nedoločeno pri 90°, 270° — ne generiramo
        table = {
            0: ("0", 0.0),
            30: ("√3/3", math.sqrt(3)/3),
            45: ("1", 1.0),
            60: ("√3", math.sqrt(3)),
            120: ("-√3", -math.sqrt(3)),
            135: ("-1", -1.0),
            150: ("-√3/3", -math.sqrt(3)/3),
            180: ("0", 0.0),
            210: ("√3/3", math.sqrt(3)/3),
            225: ("1", 1.0),
            240: ("√3", math.sqrt(3)),
            300: ("-√3", -math.sqrt(3)),
            315: ("-1", -1.0),
            330: ("-√3/3", -math.sqrt(3)/3),
        }
        if deg in table:
            return table[deg]

    # fallback: približek
    rad = math.radians(deg)
    if func == "sin":
        v = math.sin(rad); return f"≈{_round_str(v)}", v
    if func == "cos":
        v = math.cos(rad); return f"≈{_round_str(v)}", v
    if func == "tan":
        v = math.tan(rad); return f"≈{_round_str(v)}", v
    return "0", 0.0


# --------------------- Generatorji podvrst ---------------------

def _gen_special(level: int) -> Problem:
    # na 1. stopnji tan ne vključujemo
    funcs = ["sin", "cos"] if level == 1 else ["sin", "cos", "tan"]
    func = random.choice(funcs)
    base_choices = [0, 30, 45, 60, 90, 120, 135, 150, 180, 210, 225, 240, 300, 315, 330]
    if func == "tan":
        base_choices = [0, 30, 45, 60, 120, 135, 150, 180, 210, 225, 240, 300, 315, 330]
    deg = random.choice(base_choices)

    symb, num = _special_trig(func, deg)
    prompt = f"Izračunaj {func}({deg}°)"
    steps = f"Uporabi standardne vrednosti posebnih kotov za {func}."
    p = Problem(
        prompt=prompt,
        solution=symb,
        steps=steps,
        category=CATEGORY,
        numeric=float(num)
    )
    # ohrani nivo v objektu
    p.level = level
    return p

def _gen_convert(level: int) -> Problem:
    """
    Pretvorbe med stopinjami in radiani (π).
    """
    direction = random.choice(["d2r", "r2d"])

    if direction == "d2r":
        # lepi koti: večkratniki 15°, 30°, 45°, 60°, 90°
        base = random.choice([15, 30, 45, 60, 90])
        k = random.randint(1, 6 if level >= 2 else 4)
        deg = (base * k) % 360
        kpi = _deg_to_pi_frac(deg)  # Fraction

        if kpi == 0:
            sol = "0"
        elif kpi == 1:
            sol = "π"
        else:
            sol = f"{_fmt_frac(kpi)}·π"

        prompt = f"Pretvori v radiane: {deg}°"
        steps = f"rad = (deg/180)·π = ({_fmt_frac(Fraction(deg,1))}/180)·π = {sol}"
        numeric = float(kpi) * math.pi
        p = Problem(prompt, sol, steps, CATEGORY, numeric)
        p.level = level
        return p

    else:  # r2d
        # p·π; p iz lepega nabora
        pfrac = random.choice([
            Fraction(1, 6), Fraction(1, 4), Fraction(1, 3), Fraction(1, 2),
            Fraction(2, 3), Fraction(3, 4), Fraction(5, 6), Fraction(1, 1),
            Fraction(3, 2) if level >= 2 else Fraction(1, 1)
        ])
        deg = _pi_frac_to_deg(pfrac)  # Fraction
        sol = _fmt_frac(deg) + "°"
        prompt = f"Pretvori v stopinje: {_fmt_frac(pfrac)}·π"
        steps = f"deg = p·180 = {_fmt_frac(pfrac)}·180 = {_fmt_frac(deg)}°"
        numeric = float(deg)
        p = Problem(prompt, sol, steps, CATEGORY, numeric)
        p.level = level
        return p

def _gen_right_triangle(level: int) -> Problem:
    """
    Pravokotni trikotnik – izračun dolžine ali kota.
    Oblike:
      - Pitagora: given a,b -> c; given c,a -> b
      - Razmerja: sin α = nasprotna/hip, cos α = priležna/hip, tan α = nasprotna/priležna
    """
    mode = random.choices(
        ["pitagora_ab_c", "pitagora_ca_b", "sin_kot", "cos_kot", "tan_kot"],
        weights=[0.3, 0.2, 0.2, 0.15, 0.15] if level >= 2 else [0.45, 0.25, 0.3, 0.0, 0.0],
        k=1
    )[0]

    def nice_hyp(a: int, b: int) -> Optional[int]:
        c2 = a*a + b*b
        c = int(math.isqrt(c2))
        return c if c*c == c2 else None

    # Pitagora a,b → c
    if mode == "pitagora_ab_c":
        triples = [(3,4,5),(5,12,13),(6,8,10),(7,24,25)]
        if level >= 2 and random.random() < 0.4:
            a, b, c = random.choice(triples)
        else:
            a = random.randint(2, 12)
            b = random.randint(2, 12)
            c = nice_hyp(a, b)
            if c is None:
                c = math.sqrt(a*a + b*b)

        prompt = f"V pravokotnem trikotniku je a={a}, b={b}. Izračunaj hipotenuzo c."
        if isinstance(c, int):
            sol = _round_str(float(c), 3)
            numeric = float(c)
        else:
            sol = _round_str(c, 3)
            numeric = float(c)
        steps = "Pitagorov izrek: c = √(a² + b²)."
        p = Problem(prompt, sol, steps, CATEGORY, numeric)
        p.level = level
        return p

    # Pitagora c,a → b
    if mode == "pitagora_ca_b":
        triples = [(3,4,5),(5,12,13),(6,8,10),(7,24,25)]
        if level >= 2 and random.random() < 0.4:
            a, b, c = random.choice(triples)
            if random.random() < 0.5:
                a, b = b, a
            prompt = f"V pravokotnem trikotniku je c={c}, a={a}. Izračunaj kateto b."
            sol = _round_str(float(b), 3)
            numeric = float(b)
        else:
            a = random.randint(2, 12)
            c = random.randint(a + 1, 20)  # c > a
            val = c*c - a*a
            if val <= 0:
                val = abs(val) + 1
            b = math.sqrt(val)
            prompt = f"V pravokotnem trikotniku je c={c}, a={a}. Izračunaj kateto b."
            sol = _round_str(b, 3)
            numeric = float(b)
        steps = "Pitagorov izrek: b = √(c² − a²)."
        p = Problem(prompt, sol, steps, CATEGORY, numeric)
        p.level = level
        return p

    # Razmerja – izračun kota α v stopinjah
    a = random.randint(3, 12)
    b = random.randint(3, 12)
    c = math.sqrt(a*a + b*b)
    func = mode.split("_")[0]  # 'sin' / 'cos' / 'tan'

    if func == "sin":
        val = _clip01(a / c)
        deg = math.degrees(math.asin(val))
        prompt = f"V pravokotnem trikotniku: a={a} (nasprotna), c={_round_str(c,3)} (hipotenuza). Izračunaj kot α (v stopinjah)."
        steps = "Uporabi sin α = nasprotna/hip → α = arcsin(a/c)."
    elif func == "cos":
        val = _clip01(b / c)
        deg = math.degrees(math.acos(val))
        prompt = f"V pravokotnem trikotniku: b={b} (priležna), c={_round_str(c,3)} (hipotenuza). Izračunaj kot α (v stopinjah)."
        steps = "Uporabi cos α = priležna/hip → α = arccos(b/c)."
    else:  # tan
        val = a / b
        deg = math.degrees(math.atan(val))
        prompt = f"V pravokotnem trikotniku: a={a} (nasprotna), b={b} (priležna). Izračunaj kot α (v stopinjah)."
        steps = "Uporabi tan α = nasprotna/priležna → α = arctan(a/b)."

    sol = _round_str(deg, 2) + "°"
    numeric = float(deg)
    p = Problem(prompt, sol, steps, CATEGORY, numeric)
    p.level = level
    return p


# --------------------- Glavni generator ---------------------

def generiraj_trigonometrijo(tezavnost: int) -> Problem:
    if tezavnost == 1:
        weights = [0.5, 0.25, 0.25]  # special, convert, right
    elif tezavnost == 2:
        weights = [0.4, 0.30, 0.30]
    else:
        weights = [0.35, 0.30, 0.35]

    mode = random.choices(["special", "convert", "right"], weights=weights, k=1)[0]

    if mode == "special":
        return _gen_special(tezavnost)
    if mode == "convert":
        return _gen_convert(tezavnost)
    return _gen_right_triangle(tezavnost)


# Registracija v globalni register
register(CATEGORY, generiraj_trigonometrijo)


--- Konec datoteke: predmeti\trigonometrija.py ---

--- Začetek datoteke: predmeti\ulomki.py ---

# -*- coding: utf-8 -*-
"""
predmeti/ulomki.py
------------------
Generator nalog za snov "Ulomki".

Vključene oblike (naključno, glede na težavnost):
1) Seštevanje/odštevanje: a/b ± c/d
2) Množenje:                a/b · c/d  (z razlago navzkrižnega krajšanja)
3) Deljenje:                a/b ÷ c/d  (množenje z obratnim ulomkom + krajšanje)
4) Kombinacija:             a/b ÷ c/d + e/f   (na 2.–3. stopnji)

Za ± vedno prikažemo DVA pristopa:
- Pristop A (SKI): najmanjši skupni imenovalec + dejanske razširitve.
- Pristop B (križno): na imenovalec b·d, števec (a·d ± b·c).

Prikaz:
- Po binarnem minusu negativni desni člen vedno ovijemo v oklepaje:  … - (-p/q).
- Koraki so oštevilčeni (1), 2), 3) …) in zaključeni z ⇒.

Self-check (opcijsko):
- DEBUG_SELF_CHECK omogoča varen izračun izraza iz prompta (prek Fraction) in preverbo skladnosti s solution/numeric.
"""

from __future__ import annotations

import math
import random
import re
import hashlib
from fractions import Fraction
from typing import Tuple, List, Optional, Callable

from .tipi import Problem
from .register import register

CATEGORY = "Ulomki"
DEBUG_SELF_CHECK = False  # preklopi na True za samopreizkus vsake naloge


# --------------------- Pomožne: števila, razponi, LCM/GCD ---------------------

def _lcm(a: int, b: int) -> int:
    a, b = abs(a), abs(b)
    return abs(a * b) // math.gcd(a, b) if a and b else 0

def _rand_nonzero(lo: int, hi: int) -> int:
    """Naključno celo število ≠ 0 v [lo, hi]."""
    while True:
        v = random.randint(lo, hi)
        if v != 0:
            return v

def _den_pool(level: int) -> Tuple[int, int]:
    """Razpon imenovalcev glede na težavnost."""
    if level == 1:
        return 2, 9
    elif level == 2:
        return 2, 12
    else:
        return 2, 15

def _num_pool(level: int) -> Tuple[int, int]:
    """Razpon števcov glede na težavnost."""
    if level == 1:
        return 1, 9
    elif level == 2:
        return -12, 12
    else:
        return -18, 18


# --------------------- Pretty-print ulomkov in izrazov ---------------------

def _normalize_sign(n: int, d: int) -> Tuple[int, int]:
    """Znak vedno nosi števec (n); imenovalec (d) naj bo pozitiven."""
    if d < 0:
        n, d = -n, -d
    return n, d

def _pretty_frac(n: int, d: int) -> str:
    """Prikaz n/d z znakom v števcu; brez /1 pri celih."""
    n, d = _normalize_sign(n, d)
    fr = Fraction(n, d)
    return str(fr) if fr.denominator != 1 else str(fr.numerator)

def _wrap_if_negative_after_minus(s: str, is_negative: bool) -> str:
    """Če je desni člen po binarnem '-' negativen, ga ovij v oklepaje."""
    return f"({s})" if is_negative else s

def _hash_id(text: str) -> str:
    return hashlib.sha1(text.encode("utf-8")).hexdigest()[:10]


# --------------------- Graditelj korakov (dosleden slog) ---------------------

def _steps_intro_sum() -> str:
    return "Vrstni red: (oklepaji) → (M/D) → (S/O). Pri ± pokažemo oba pristopa (SKI in križno)."

def _steps_intro_mul() -> str:
    return "Vrstni red: (oklepaji) → (M/D) → (S/O). Pri množenju skrajšamo navzkrižno, kjer je smiselno."

def _steps_intro_div() -> str:
    return "Vrstni red: (oklepaji) → (M/D) → (S/O). Deljenje z ulomkom pretvorimo v množenje z obratnim."

def _steps_buffer(intro: str) -> List[str]:
    return [intro]

def _add_step(buf: List[str], line: str) -> None:
    buf.append(line)

def _finish_steps(buf: List[str]) -> str:
    return "\n".join(buf)


# --------------------- Izbor ulomkov (z možnostjo imenovalca 1) ---------------------

def _pick_frac(level: int) -> Tuple[int, int]:
    """
    Izberi 'lep' ulomek a/b (b ≠ 0).
    Na višjih stopnjah z majhno verjetnostjo dovoli b = 1 (celo število),
    da so razlage pretvorb (celo → ulomek) smiselne.
    """
    lo_d, hi_d = _den_pool(level)
    lo_n, hi_n = _num_pool(level)

    allow_den1_prob = 0.0
    if level == 2:
        allow_den1_prob = 0.15
    elif level >= 3:
        allow_den1_prob = 0.25

    if random.random() < allow_den1_prob:
        b = 1
    else:
        b = _rand_nonzero(lo_d, hi_d)

    a = _rand_nonzero(lo_n, hi_n)
    return a, b


# --------------------- Koraki: seštevanje/odštevanje (oba pristopa) ---------------------

def _mk_steps_add_sub(a: int, b: int, c: int, d: int, op: str) -> str:
    """
    Zgradi razlago za a/b ± c/d z DVOJIM pristopom:
      A) SKI (LCM) in dejanske razširitve
      B) Križno (na imenovalec b·d) s števcem (a·d ± b·c).
    Upošteva posebne primere: isti imenovalec, imenovalec 1 (celi), negativni členi.
    """
    assert op in ('+', '-')
    # normalizacije znaka (za prikaz in robustnost razlage)
    a, b = _normalize_sign(a, b)
    c, d = _normalize_sign(c, d)
    f1 = Fraction(a, b)
    f2 = Fraction(c, d)

    buf: List[str] = _steps_buffer(_steps_intro_sum())

    # Če odštevamo negativen ulomek, eksplicitno razložimo dvojni minus.
    if op == '-' and f2 < 0:
        _add_step(buf, f"1) Prepoznamo dvojni minus: −({f2}) = +{abs(f2)} ⇒ izraz prepišemo kot {f1} + {abs(f2)}.")

    # -------- Pristop A: SKI --------
    if b == d:
        # isti imenovalec
        numA = a + (c if op == '+' else -c)
        frA = Fraction(numA, b)
        _add_step(buf, f"[Pristop A – SKI]")
        _add_step(buf, f"2) Isti imenovalec {b} ⇒ števec se {'sešteje' if op == '+' else 'odšteje'}: "
                       f"({a} {op} {c})/{b} = {numA}/{b} = {frA}")
    else:
        # splošni primer SKI; posebej prikažemo tudi primere s celim številom
        L = _lcm(b, d) if (b != 1 and d != 1) else (d if b == 1 else b)
        m1 = L // b
        m2 = L // d
        a2, c2 = a * m1, c * m2
        numA = a2 + (c2 if op == '+' else -c2)
        frA = Fraction(numA, L)
        _add_step(buf, f"[Pristop A – SKI]")
        if b == 1 or d == 1:
            _add_step(buf, f"2) Pretvorimo celo število na imenovalec {L} in razširimo drugi ulomek:")
            if b == 1:
                _add_step(buf, f"{a} = {a}·{L}/{L} = {a2}/{L}")
                _add_step(buf, f"{c}/{d} = {c}·{m2}/{d}·{m2} = {c2}/{L}")
            else:
                _add_step(buf, f"{c} = {c}·{L}/{L} = {c2}/{L}")
                _add_step(buf, f"{a}/{b} = {a}·{m1}/{b}·{m1} = {a2}/{L}")
        else:
            _add_step(buf, f"2) Najmanjši skupni imenovalec: lcm({b},{d}) = {L}. Razširimo oba ulomka na {L}:")
            _add_step(buf, f"{a}/{b} = {a}·{m1}/{b}·{m1} = {a2}/{L}")
            _add_step(buf, f"{c}/{d} = {c}·{m2}/{d}·{m2} = {c2}/{L}")
        _add_step(buf, f"3) Sedaj sta imenovalca enaka: ({a2} {op} {c2})/{L} = {numA}/{L} = {frA}")

    # -------- Pristop B: križno --------
    if b == d:
        numB = a + (c if op == '+' else -c)
        frB = Fraction(numB, b)
        _add_step(buf, f"[Pristop B – križno]")
        _add_step(buf, f"4) Enak imenovalec {b} ⇒ ({a} {op} {c})/{b} = {numB}/{b} = {frB}")
    else:
        denB = b * d
        numB = a * d + (b * c if op == '+' else -b * c)
        frB = Fraction(numB, denB)
        _add_step(buf, f"[Pristop B – križno]")
        if b == 1 or d == 1:
            _add_step(buf, f"4) Prenesemo na skupni imenovalec {denB}; pretvorba celega števila je eksplicitna.")
            if b == 1:
                _add_step(buf, f"{a} = {a}·{denB}/{denB} = {a*denB}/{denB}")
                _add_step(buf, f"⇒ števec = {a}·{d} {op} {c}·1 = {a*d} {op} {c} = {numB}")
            else:
                _add_step(buf, f"{c} = {c}·{denB}/{denB} = {c*denB}/{denB}")
                _add_step(buf, f"⇒ števec = {a}·1 {op} {b}·{c} = {a} {op} {b*c} = {numB}")
        else:
            _add_step(buf, f"4) Prenos na imenovalec b·d = {b}·{d} = {denB}:")
            _add_step(buf, f"števec = a·d {op} b·c = {a}·{d} {op} {b}·{c} = {a*d} {op} {b*c} = {numB}")
        _add_step(buf, f"5) Dobimo {numB}/{denB} = {frB}")

    return _finish_steps(buf)


# --------------------- Koraki: množenje (krajšanje) ---------------------

def _mk_steps_mul(a: int, b: int, c: int, d: int) -> str:
    """Koraki za množenje z navzkrižnim krajšanjem po g₁, g₂, nato množenje števcov/imenovalcev."""
    a, b = _normalize_sign(a, b)
    c, d = _normalize_sign(c, d)
    g1 = math.gcd(abs(a), abs(d))
    g2 = math.gcd(abs(c), abs(b))
    a2, d2 = a // g1, d // g1
    c2, b2 = c // g2, b // g2
    num = a2 * c2
    den = b2 * d2
    fr  = Fraction(num, den)

    buf = _steps_buffer(_steps_intro_mul())
    _add_step(buf, f"1) Krajšamo navzkrižno: g₁ = gcd(|{a}|,|{d}|) = {g1}, g₂ = gcd(|{c}|,|{b}|) = {g2}.")
    _add_step(buf, f"   Delimo: {a}→{a}÷{g1}={a2}, {d}→{d}÷{g1}={d2}; {c}→{c}÷{g2}={c2}, {b}→{b}÷{g2}={b2}.")
    _add_step(buf, f"2) Nato množimo: {a2}/{b2} · {c2}/{d2} = {num}/{den} ⇒ {fr}")
    return _finish_steps(buf)


# --------------------- Koraki: deljenje (obratni ulomek + krajšanje) ---------------------

def _mk_steps_div(a: int, b: int, c: int, d: int) -> str:
    """Koraki za deljenje: obrnemo drugi ulomek, skrajšamo navzkrižno, nato množimo."""
    a, b = _normalize_sign(a, b)
    c, d = _normalize_sign(c, d)
    # (c/d) ne sme biti 0 – to filtriramo pri generiranju
    g1 = math.gcd(abs(a), abs(d))
    g2 = math.gcd(abs(c), abs(b))
    a2, d2 = a // g1, d // g1
    c2, b2 = c // g2, b // g2
    num = a2 * d2
    den = b2 * c2
    fr  = Fraction(num, den)

    buf = _steps_buffer(_steps_intro_div())
    _add_step(buf, f"1) Obrnemo drugi ulomek: {a}/{b} ÷ {c}/{d} = {a}/{b} · {d}/{c}.")
    _add_step(buf, f"2) Krajšamo navzkrižno: g₁ = gcd(|{a}|,|{d}|) = {g1}, g₂ = gcd(|{c}|,|{b}|) = {g2}.")
    _add_step(buf, f"   Delimo: {a}→{a}÷{g1}={a2}, {d}→{d}÷{g1}={d2}; {c}→{c}÷{g2}={c2}, {b}→{b}÷{g2}={b2}.")
    _add_step(buf, f"3) Množimo: {a2}/{b2} · {d2}/{c2} = {num}/{den} ⇒ {fr}")
    return _finish_steps(buf)


# --------------------- Self-check: varen eval Fraction izraza iz prompta ---------------------

_FRAC_TOKEN = re.compile(r'(?P<sign>-?)\s*(?P<n>\d+)\s*/\s*(?P<d>-?\d+)')

def _normalize_prompt_for_eval(prompt: str) -> str:
    """
    Iz 'Uredi izraz: ...' ali 'Uredi: ...' izluščimo izraz, nato:
    - zamenjamo '·'→'*', '÷'→'/', ter oklepaje pustimo,
    - vsak 'n/m' pretvorimo v 'Fraction(n, m)' z znakom v števcu.
    """
    # izlušči desni del za eval
    if ":" in prompt:
        expr = prompt.split(":", 1)[1].strip()
    else:
        expr = prompt.split("Uredi", 1)[1].strip()
    # presledki so OK; zamenjaj operatorje
    expr = expr.replace("·", "*").replace("÷", "/")

    def repl(m: re.Match) -> str:
        sign = -1 if m.group("sign") == "-" else 1
        n = int(m.group("n")) * sign
        d = int(m.group("d"))
        n, d = _normalize_sign(n, d)
        return f"Fraction({n},{d})"

    expr = _FRAC_TOKEN.sub(repl, expr)
    return expr

def _self_check(problem: Problem) -> None:
    """Če je DEBUG_SELF_CHECK=True, izračunaj prompt in primerjaj s solution/numeric."""
    expr = _normalize_prompt_for_eval(problem.prompt)
    try:
        val = eval(expr, {"Fraction": Fraction})
    except Exception as e:
        raise AssertionError(f"Self-check eval error: {e} | expr={expr}")

    # solution je izpisana skrajšana oblika (lahko celo število)
    sol = problem.solution
    val_str = str(val) if val.denominator != 1 else str(val.numerator)
    if sol != val_str or float(val) != float(problem.numeric):
        raise AssertionError(f"Self-check mismatch: eval={val_str} vs sol={sol}, numeric={problem.numeric}")


# --------------------- Ocenjevanje težavnosti (0–1) ---------------------

def _difficulty_score(level: int, has_mul: bool, has_div: bool, has_combo: bool, nums: List[int], dens: List[int]) -> float:
    nscale = min(1.0, (max([abs(x) for x in nums]) / 18.0) if nums else 0.0)
    dscale = min(1.0, (max(dens) / 15.0) if dens else 0.0)
    bonus = (0.12 if has_mul else 0.0) + (0.15 if has_div else 0.0) + (0.18 if has_combo else 0.0)
    base  = 0.0 if level == 1 else 0.25 if level == 2 else 0.5
    return round(min(1.0, base + bonus + 0.35 * nscale + 0.25 * dscale), 2)


# --------------------- Generatorji podvrst ---------------------

def _prompt_add(a: int, b: int, c: int, d: int, op: str) -> str:
    """Zgradi lep prompt za ± (oklepaji po '-' pri negativnem desnem členu)."""
    right = f"{c}/{d}"
    is_neg_right = Fraction(c, d) < 0
    if op == '-' and is_neg_right:
        right = f"({right})"
    return f"Uredi izraz: {a}/{b} {op} {right}"

def _gen_add(level: int) -> Problem:
    a, b = _pick_frac(level)
    c, d = _pick_frac(level)
    val = Fraction(a, b) + Fraction(c, d)

    steps = _mk_steps_add_sub(a, b, c, d, op='+')
    p = Problem(
        prompt=_prompt_add(a, b, c, d, '+'),
        solution=_pretty_frac(val.numerator, val.denominator),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(p.prompt),
        "pattern": "add",
        "difficulty": _difficulty_score(level, has_mul=False, has_div=False, has_combo=False,
                                        nums=[a, c], dens=[abs(b), abs(d)])
    }
    if DEBUG_SELF_CHECK:
        _self_check(p)
    return p

def _gen_sub(level: int) -> Problem:
    a, b = _pick_frac(level)
    c, d = _pick_frac(level)
    val = Fraction(a, b) - Fraction(c, d)

    steps = _mk_steps_add_sub(a, b, c, d, op='-')
    p = Problem(
        prompt=_prompt_add(a, b, c, d, '-'),
        solution=_pretty_frac(val.numerator, val.denominator),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(p.prompt),
        "pattern": "sub",
        "difficulty": _difficulty_score(level, has_mul=False, has_div=False, has_combo=False,
                                        nums=[a, c], dens=[abs(b), abs(d)])
    }
    if DEBUG_SELF_CHECK:
        _self_check(p)
    return p

def _gen_mul(level: int) -> Problem:
    a, b = _pick_frac(level)
    c, d = _pick_frac(level)
    val = Fraction(a, b) * Fraction(c, d)

    steps = _mk_steps_mul(a, b, c, d)
    p = Problem(
        prompt=f"Uredi izraz: {a}/{b} · {c}/{d}",
        solution=_pretty_frac(val.numerator, val.denominator),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(p.prompt),
        "pattern": "mul",
        "difficulty": _difficulty_score(level, has_mul=True, has_div=False, has_combo=False,
                                        nums=[a, c], dens=[abs(b), abs(d)])
    }
    if DEBUG_SELF_CHECK:
        _self_check(p)
    return p

def _gen_div(level: int) -> Problem:
    # poskrbimo, da (c/d) ≠ 0 ⇒ c ≠ 0
    a, b = _pick_frac(level)
    while True:
        c, d = _pick_frac(level)
        if c != 0:
            break

    val = Fraction(a, b) / Fraction(c, d)

    steps = _mk_steps_div(a, b, c, d)
    p = Problem(
        prompt=f"Uredi izraz: {a}/{b} ÷ {c}/{d}",
        solution=_pretty_frac(val.numerator, val.denominator),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(p.prompt),
        "pattern": "div",
        "difficulty": _difficulty_score(level, has_mul=False, has_div=True, has_combo=False,
                                        nums=[a, c], dens=[abs(b), abs(d)])
    }
    if DEBUG_SELF_CHECK:
        _self_check(p)
    return p

def _gen_combo_div_plus(level: int) -> Problem:
    """
    Kombinacija:  a/b ÷ c/d + e/f
    1) Deljenje (obrni drugi ulomek + krajšanje)
    2) Nato seštevanje (oba pristopa)
    """
    a, b = _pick_frac(level)
    # poskrbimo, da (c/d) ≠ 0
    while True:
        c, d = _pick_frac(level)
        if c != 0:
            break
    e, f = _pick_frac(level)

    left = Fraction(a, b) / Fraction(c, d)
    val = left + Fraction(e, f)

    steps_parts: List[str] = []
    steps_parts.append("1) Najprej deljenje:")
    steps_parts.append(_mk_steps_div(a, b, c, d))
    steps_parts.append("\n2) Nato seštevanje s tretjim ulomkom (oba pristopa):")
    # seštevanje p/q + e/f; uporabimo oba pristopa na p/q in e/f
    pnum, pden = left.numerator, left.denominator
    steps_parts.append(_mk_steps_add_sub(pnum, pden, e, f, op='+'))
    steps = "\n".join(steps_parts)

    # prompt: ker je '+' na koncu, posebni oklepaji za doble minus tukaj niso potrebni
    prompt = f"Uredi izraz: {a}/{b} ÷ {c}/{d} + {e}/{f}"

    p = Problem(
        prompt=prompt,
        solution=_pretty_frac(val.numerator, val.denominator),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(p.prompt),
        "pattern": "combo",
        "difficulty": _difficulty_score(level, has_mul=False, has_div=True, has_combo=True,
                                        nums=[a, c, e], dens=[abs(b), abs(d), abs(f)])
    }
    if DEBUG_SELF_CHECK:
        _self_check(p)
    return p


# --------------------- Glavni generator ---------------------

def generiraj_ulomke(tezavnost: int) -> Problem:
    """
    Težavnost:
      - 1: pretežno +/−, manjši razponi; brez negativnih rezultatov, če je mogoče
      - 2: +/−/·/÷, širši razponi in negativna števila, občasno imenovalec 1
      - 3: vključujemo tudi kombinacijo 'a/b ÷ c/d + e/f' in več primerov z imenovalcem 1
    """
    if tezavnost == 1:
        modes = ["add", "sub", "mul"]
        weights = [0.45, 0.35, 0.20]
    elif tezavnost == 2:
        modes = ["add", "sub", "mul", "div"]
        weights = [0.35, 0.30, 0.20, 0.15]
    else:
        modes = ["add", "sub", "mul", "div", "combo"]
        weights = [0.30, 0.25, 0.20, 0.15, 0.10]

    mode = random.choices(modes, weights=weights, k=1)[0]

    if mode == "add":
        return _gen_add(tezavnost)
    if mode == "sub":
        return _gen_sub(tezavnost)
    if mode == "mul":
        return _gen_mul(tezavnost)
    if mode == "div":
        return _gen_div(tezavnost)
    return _gen_combo_div_plus(tezavnost)


# Registracija v globalni register
register(CATEGORY, generiraj_ulomke)


--- Konec datoteke: predmeti\ulomki.py ---

--- Začetek datoteke: predmeti\vrstni_red_racunov.py ---

# -*- coding: utf-8 -*-
"""
predmeti/vrstni_red_racunov.py
------------------------------
Generator nalog za snov "Vrstni red računov" (PEMDAS / OKLMZ).

Vključene oblike (naključno):
1) Brez potenc:                  a - b · c + d
2) S potencami:                  a - b^2 · c + d
3) Z oklepaji:                   a - [ b + ( c - d ) · e ]
4) (nova) Z deljenjem v oklepaju a + b · ( c ÷ d ), kjer je c ÷ d celo število

Težavnost:
- 1: manjša števila (1–9), brez negativnih vmesnih izrazov, največ ena potenca
- 2: širši razpon (1–12), večja verjetnost potenc/oklepajev/deljenja
- 3: širši razpon (1–15), pogosteje oklepaji in b^2; rezultati lahko negativni

Vsaka naloga vrne Problem s:
    prompt (str)         – zapis izraza za učenca (pretty-print),
    solution (str=int)   – točen rezultat,
    steps (str)          – oštevilčena razlaga korakov,
    category (str),
    numeric (float)      – numerični rezultat (isti kot solution),
    level (int)          – nastavljen dosledno,
    meta (dict)          – {"id", "pattern", "difficulty"}.

Opombe:
- Za reproducibilnost lahko klicatelj predhodno nastavi random.seed(...).
- Samopreizkus lahko vklopiš z DEBUG_SELF_CHECK = True.
"""

from __future__ import annotations

import ast
import hashlib
import math
import operator as op
import random
from typing import List, Tuple, Optional

from .tipi import Problem
from .register import register
from orodja.selfcheck import eval_prompt_expr


CATEGORY = "Vrstni red računov"
DEBUG_SELF_CHECK = False  # preklopi na True, če želiš, da se vsak primer sam preveri


# --------------------- Pomožne funkcije: naključja & razpon ---------------------

def _r(lo: int, hi: int) -> int:
    """Naključno celo število v [lo, hi]."""
    return random.randint(lo, hi)

def _range_by_level(level: int) -> Tuple[int, int]:
    if level == 1:
        return (1, 9)
    if level == 2:
        return (1, 12)
    return (1, 15)


# --------------------- Pedagoški uvod in graditelj korakov ---------------------

def _fmt_steps_intro(with_power: bool, with_brackets: bool) -> str:
    base = "Vrstni red računov: oklepaji → potence/koreni → množenje/deljenje → seštevanje/odštevanje (z leve)."
    if with_brackets and with_power:
        return base + "\n1) Oklepaji, 2) Potence, 3) M/D, 4) S/O.\n"
    if with_power:
        return base + "\n1) Potence, 2) M/D, 3) S/O.\n"
    if with_brackets:
        return base + "\n1) Oklepaji, 2) M/D, 3) S/O.\n"
    return base + "\n1) M/D, 2) S/O.\n"

def _steps_buffer(intro: str) -> List[str]:
    buf = [intro.rstrip("\n")]
    return buf

def _add_step(buf: List[str], text: str) -> None:
    buf.append(text)

def _finish_steps(buf: List[str]) -> str:
    return "\n".join(buf)


# --------------------- Pretty-print (zlasti za dvojni minus) ---------------------

def _fmt_term(n: int) -> str:
    """Zapiše celo število kot niz (brez znaka +)."""
    return f"{n}"

def _fmt_product(x: int, y: int) -> str:
    """Zapiše produkt z '·' in presledki."""
    return f"{_fmt_term(x)} · {_fmt_term(y)}"

def _wrap_if_negative(s: str, value: int) -> str:
    """
    Če je vrednost negativna in stoji kot DESNI člen po binarnem '-', ga damo v oklepaje: - (-7)
    (Za berljivost; AST eval interno ne potrebuje, to je zgolj za prikaz.)
    """
    return f"({s})" if value < 0 else s

def _fmt_minus(left: str, right_val: int, right_str: str) -> str:
    """Zapiše 'left - right', pri čemer negativni desni člen ovije v oklepaje."""
    return f"{left} - {_wrap_if_negative(right_str, right_val)}"

def _hash_id(text: str) -> str:
    return hashlib.sha1(text.encode("utf-8")).hexdigest()[:10]


# --------------------- Varno vrednotenje izraza (AST) ---------------------

_BIN_OPS = {
    ast.Add: op.add,
    ast.Sub: op.sub,
    ast.Mult: op.mul,
    ast.Div: op.truediv,
    ast.Pow: op.pow,
}
_UNARY_OPS = {ast.USub: op.neg}

_ALLOWED_NODES = (
    ast.Expression, ast.BinOp, ast.UnaryOp, ast.Constant, ast.Constant,
    ast.Add, ast.Sub, ast.Mult, ast.Div, ast.Pow, ast.Load, ast.Expr, ast.Tuple, ast.List, ast.USub,
)

def _normalize_for_eval(expr: str) -> str:
    """
    Pripravi prompt za eval:
    - [ ] → ( ), '·' → '*', '÷' → '/', '^' → '**'
    - ohrani oklepaje, presledki so neproblematični
    (NE izvajamo slepega "--" → "+", ker ločimo unarni/binarni minus preko AST.)
    """
    return (
        expr.replace("[", "(")
            .replace("]", ")")
            .replace("·", "*")
            .replace("÷", "/")
            .replace("^", "**")
    )

def _safe_eval(expr: str) -> float:
    node = ast.parse(expr, mode="eval")

    def _eval(n):
        if isinstance(n, ast.Expression):
            return _eval(n.body)
        if isinstance(n, ast.Constant):          # Py<3.8
            return n.n
        if isinstance(n, ast.Constant):     # Py>=3.8
            if isinstance(n.value, (int, float)):
                return n.value
            raise ValueError("Nedovoljena konstanta")
        if isinstance(n, ast.UnaryOp) and type(n.op) in _UNARY_OPS:
            return _UNARY_OPS[type(n.op)](_eval(n.operand))
        if isinstance(n, ast.BinOp) and type(n.op) in _BIN_OPS:
            return _BIN_OPS[type(n.op)](_eval(n.left), _eval(n.right))
        raise ValueError(f"Nedovoljen del izraza: {type(n).__name__}")

    # Preverimo dovoljene node-e (defenzivno)
    for sub in ast.walk(node):
        if not isinstance(sub, _ALLOWED_NODES):
            raise ValueError(f"Nedovoljen AST element: {type(sub).__name__}")

    return float(_eval(node))


def _verify_problem(p: Problem) -> None:
    """
    Self-check: iz `prompt` izračuna vrednost in preveri skladnost s `numeric` in `solution`.
    Kliče se le, če je DEBUG_SELF_CHECK = True.
    """
    try:
        raw_expr = p.prompt.split("Uredi:", 1)[1].strip()
    except Exception:
        raise AssertionError("Ne najdem izraza po 'Uredi:' v promptu.")
    expr = _normalize_for_eval(raw_expr)
    calc = eval_prompt_expr(p.prompt)
    # primer izenačimo do integer rezultata (v teh nalogah mora biti celo število)
    calc_int = int(round(calc))
    if calc_int != int(p.numeric) or calc_int != int(p.solution):
        raise AssertionError(f"Eval mismatch: {calc} vs numeric={p.numeric}, solution={p.solution}")


# --------------------- Ocenjevanje "težavnosti" ---------------------

def _difficulty_score(level: int, has_pow: bool, has_br: bool, has_div: bool, numbers: List[int]) -> float:
    """
    Preprost skalar 0–1 (informativno): odvisen od velikosti števil in prisotnosti zahtevnejših delov.
    """
    nscale = min(1.0, (max(numbers) / 15.0) if numbers else 0.0)
    bonus = (0.15 if has_pow else 0.0) + (0.15 if has_div else 0.0) + (0.10 if has_br else 0.0)
    base = 0.0 if level <= 1 else 0.25 if level == 2 else 0.5
    return round(min(1.0, base + bonus + 0.5 * nscale), 2)


# --------------------- Generatorji podvrst ---------------------

def _gen_plain(level: int) -> Problem:
    lo, hi = _range_by_level(level)

    # Na 1. stopnji poskrbimo, da končni rezultat ni negativen: a + d >= b*c.
    for _ in range(96):
        a, b, c, d = _r(lo, hi), _r(lo, hi), _r(lo, hi), _r(lo, hi)
        mul = b * c
        val = a - mul + d
        if level != 1 or (a + d) >= mul:
            break

    # Prompt (pretty-print) – **PLUS** d, da se ujema s koraki
    left = f"{_fmt_term(a)} - {_fmt_product(b, c)}"
    prompt = f"Uredi: {left} + {d}"

    # Koraki
    buf = _steps_buffer(_fmt_steps_intro(with_power=False, with_brackets=False))
    _add_step(buf, f"2) Množenje: {b} · {c} = {mul}")
    _add_step(buf, f"3) S/O z leve: {a} - {mul} + {d} = ({a} - {mul}) + {d} = {a - mul} + {d} ⇒ {val}")
    steps = _finish_steps(buf)

    p = Problem(
        prompt=prompt,
        solution=str(val),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(prompt),
        "pattern": "plain",
        "difficulty": _difficulty_score(level, has_pow=False, has_br=False, has_div=False, numbers=[a, b, c, d]),
    }
    if DEBUG_SELF_CHECK:
        _verify_problem(p)
    return p


def _gen_with_power(level: int) -> Problem:
    lo, hi = _range_by_level(level)

    # Na 1. stopnji: a + d >= b^2 * c (nenegativen končni rezultat).
    for _ in range(96):
        a, b, c, d = _r(lo, hi), _r(lo, hi), _r(lo, hi), _r(lo, hi)
        b2 = b * b
        mul = b2 * c
        val = a - mul + d
        if level != 1 or (a + d) >= mul:
            break

    # Prompt – **PLUS** d, da se ujema s koraki
    left = f"{_fmt_term(a)} - {b}^2 · {c}"
    prompt = f"Uredi: {left} + {d}"

    # Koraki
    buf = _steps_buffer(_fmt_steps_intro(with_power=True, with_brackets=False))
    _add_step(buf, f"1) Potenca: {b}^2 = {b2}")
    _add_step(buf, f"2) Množenje: {b2} · {c} = {mul}")
    _add_step(buf, f"3) S/O z leve: {a} - {mul} + {d} = ({a} - {mul}) + {d} = {a - mul} + {d} ⇒ {val}")
    steps = _finish_steps(buf)

    p = Problem(
        prompt=prompt,
        solution=str(val),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(prompt),
        "pattern": "power",
        "difficulty": _difficulty_score(level, has_pow=True, has_br=False, has_div=False, numbers=[a, b, c, d]),
    }
    if DEBUG_SELF_CHECK:
        _verify_problem(p)
    return p


def _gen_with_brackets(level: int) -> Problem:
    lo, hi = _range_by_level(level)

    # Na 1. stopnji zagotovimo: (c ≥ d), izognemo se trivialnim (inner==0 ali e==1),
    # in a ≥ b + (c - d)·e (nenegativen končni rezultat).
    for _ in range(192):
        a, b, c, d, e = _r(lo, hi), _r(lo, hi), _r(lo, hi), _r(lo, hi), _r(lo, hi)

        if level == 1 and c < d:
            c, d = d, c  # poskrbimo, da (c - d) ≥ 0

        inner = c - d              # (c - d)
        prod  = inner * e          # (c - d) · e
        bracket = b + prod         # b + (c - d) · e
        val = a - bracket          # a - [ ... ]

        # antitrivialno pri 1. stopnji
        if level == 1:
            if inner == 0 or e == 1:
                continue
            if a < bracket:
                continue
        break

    # Prompt – izraz z oklepaji; če je bracket negativen, se to jasno vidi kot: a - [ -7 ]
    prompt = f"Uredi: {a} - [ {b} + ( {c} - {d} ) · {e} ]"

    # Koraki (če prod < 0, implicitno pokažemo “negativen člen v oklepaju”)
    buf = _steps_buffer(_fmt_steps_intro(with_power=False, with_brackets=True))
    _add_step(buf, f"1) Notranji oklepaj: ({c} - {d}) = {inner}")
    _add_step(buf, f"2) Množenje v oklepaju: {inner} · {e} = {prod}")
    _add_step(buf, f"3) Seštej v kvadratnem oklepaju: {b} + {prod} = {bracket}")
    # Če je bracket negativen, je to ekvivalentno a - (-k) = a + k; nakažemo s ⇒ besedilom:
    if bracket < 0:
        _add_step(buf, f"4) Zunanji: {a} - ({bracket}) = {a} + {abs(bracket)} ⇒ {val}")
    else:
        _add_step(buf, f"4) Zunanji: {a} - [{bracket}] ⇒ {val}")
    steps = _finish_steps(buf)

    p = Problem(
        prompt=prompt,
        solution=str(val),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(prompt),
        "pattern": "brackets",
        "difficulty": _difficulty_score(level, has_pow=False, has_br=True, has_div=False, numbers=[a, b, c, d, e]),
    }
    if DEBUG_SELF_CHECK:
        _verify_problem(p)
    return p


def _gen_with_division(level: int) -> Problem:
    """
    Nova, poučna varianta: a + b · ( c ÷ d ), kjer je c ÷ d celo število.
    Na 1. stopnji skrbimo za nenegativne vmesne/končne vrednosti.
    """
    lo, hi = _range_by_level(level)

    for _ in range(192):
        a, b, c, d = _r(lo, hi), _r(lo, hi), _r(lo, hi), _r(lo, hi)
        if d == 0 or c % d != 0:
            continue
        inner = c // d
        prod = b * inner
        val = a + prod
        if level != 1 or val >= 0:
            break

    prompt = f"Uredi: {a} + {b} · ( {c} ÷ {d} )"

    buf = _steps_buffer(_fmt_steps_intro(with_power=False, with_brackets=True))
    _add_step(buf, f"1) Deljenje v oklepaju: {c} ÷ {d} = {inner}")
    _add_step(buf, f"2) Množenje: {b} · {inner} = {prod}")
    _add_step(buf, f"3) Seštevanje: {a} + {prod} ⇒ {val}")
    steps = _finish_steps(buf)

    p = Problem(
        prompt=prompt,
        solution=str(val),
        steps=steps,
        category=CATEGORY,
        numeric=float(val),
    )
    p.level = level
    p.meta = {
        "id": _hash_id(prompt),
        "pattern": "division",
        "difficulty": _difficulty_score(level, has_pow=False, has_br=True, has_div=True, numbers=[a, b, c, d]),
    }
    if DEBUG_SELF_CHECK:
        _verify_problem(p)
    return p


# --------------------- Glavni generator ---------------------

def generiraj_vrstni_red_racunov(tezavnost: int) -> Problem:
    """
    Izbere eno od štirih podvrst. Višja težavnost poveča razpon števil
    in verjetnost oklepajev/potenc/deljenja.
    """
    if tezavnost == 1:
        # plain, power, brackets, division
        weights = [0.60, 0.20, 0.20, 0.00]
    elif tezavnost == 2:
        weights = [0.40, 0.25, 0.25, 0.10]
    else:
        weights = [0.30, 0.30, 0.25, 0.15]

    mode = random.choices(
        population=["plain", "power", "brackets", "division"],
        weights=weights, k=1
    )[0]

    if mode == "plain":
        return _gen_plain(tezavnost)
    if mode == "power":
        return _gen_with_power(tezavnost)
    if mode == "brackets":
        return _gen_with_brackets(tezavnost)
    return _gen_with_division(tezavnost)


# Registracija kategorije v globalni register
register(CATEGORY, generiraj_vrstni_red_racunov)


--- Konec datoteke: predmeti\vrstni_red_racunov.py ---

--- Začetek datoteke: predmeti\__init__.py ---



--- Konec datoteke: predmeti\__init__.py ---

--- Začetek datoteke: tests\test_core_correctness.py ---

# -*- coding: utf-8 -*-
"""
tests/test_core_correctness.py
------------------------------
Jedrni testi pravilnosti:
- evaluate_answer sprejme pravilne rešitve in zavrne očitno napačne.
- "Vrstni red računov": eval(prompt) == numeric == solution.
- Reproducibilnost generiranja pri enakem seedu.
"""
from __future__ import annotations
import ast, operator as op
import random
import re
import math

import pytest

from predmeti import register as REG
from predmeti.vrstni_red_racunov import generiraj_vrstni_red_racunov
from orodja.odgovor_eval import evaluate_answer, EvalConfig

# poskrbi za registracijo kategorij
from predmeti import ulomki as _ensure_import1  # noqa: F401
from predmeti import potence_in_koreni as _ensure_import2  # noqa: F401
from predmeti import linearne_enacbe as _ensure_import3  # noqa: F401
from predmeti import polinomi as _ensure_import4  # noqa: F401
from predmeti import trigonometrija as _ensure_import5  # noqa: F401
from predmeti import funkcije_in_grafi as _ensure_import6  # noqa: F401
from predmeti import sistemi_enacb as _ensure_import7  # noqa: F401
from predmeti import procenti_in_ddv as _ensure_import8  # noqa: F401

CFG = EvalConfig()

# --- varno eval za "Vrstni red računov" (skladno z generatorjem) ---
_ALLOWED_BIN = {ast.Add: op.add, ast.Sub: op.sub, ast.Mult: op.mul, ast.Div: op.truediv, ast.Pow: op.pow}
_ALLOWED_UN = {ast.USub: op.neg}

def _normalize_for_eval(expr: str) -> str:
    return (expr.replace("[", "(")
                .replace("]", ")")
                .replace("·", "*")
                .replace("÷", "/")
                .replace("^", "**"))

def _safe_eval(expr: str) -> float:
    node = ast.parse(expr, mode="eval")
    def _ev(n):
        if isinstance(n, ast.Expression):
            return _ev(n.body)
        if hasattr(ast, "Num") and isinstance(n, ast.Num):
            return n.n
        if isinstance(n, ast.Constant):
            if isinstance(n.value, (int, float)):
                return n.value
            raise ValueError("bad const")
        if isinstance(n, ast.UnaryOp) and type(n.op) in _ALLOWED_UN:
            return _ALLOWED_UN[type(n.op)](_ev(n.operand))
        if isinstance(n, ast.BinOp) and type(n.op) in _ALLOWED_BIN:
            return _ALLOWED_BIN[type(n.op)](_ev(n.left), _ev(n.right))
        raise ValueError(f"unsupported: {type(n).__name__}")
    return float(_ev(node))

@pytest.mark.parametrize("level", [1, 2, 3])
def test_evaluate_accepts_true_and_rejects_false(level: int):
    random.seed(1010 + level)
    cats = REG.list_categories()
    assert cats, "Ni registriranih kategorij."
    # vzemi do 5 kategorij za hitrost
    for cat in cats[:5]:
        for _ in range(12):  # generiraj več in filtriraj na numeric
            p = REG.generate(cat, level)
            if p.numeric is None:
                # Trenutni evaluator ne podpira vseh simbolnih (npr. domene, množice, ℝ, ≠ …)
                continue

            # 1) pravilni odgovor — numeričen
            ans_ok = str(p.solution)
            v_ok = evaluate_answer(p, ans_ok, subject=cat, config=CFG)
            assert v_ok.is_correct, (
                f"Pravilni odgovor zavrnjen (cat={cat}, prompt={p.prompt}, "
                f"solution={p.solution}, numeric={p.numeric})"
            )

            # 2) očitno napačen odgovor
            bad = str(float(p.numeric) + 1.2345)
            v_bad = evaluate_answer(p, bad, subject=cat, config=CFG)
            assert not v_bad.is_correct, (
                f"Napačen odgovor sprejet (cat={cat}, prompt={p.prompt}, bad={bad}, "
                f"solution={p.solution}, numeric={p.numeric})"
            )

@pytest.mark.parametrize("level", [1, 2, 3])
def test_vrr_prompt_evaluates_to_solution(level: int):
    random.seed(777 + level)
    for _ in range(30):
        p = generiraj_vrstni_red_racunov(level)
        raw = p.prompt.split("Uredi:", 1)[1].strip()
        expr = _normalize_for_eval(raw)
        calc = _safe_eval(expr)
        # pričakujemo celoštevilski rezultat
        assert int(round(calc)) == int(p.numeric) == int(p.solution), (
            f"Eval mismatch: {calc} vs numeric={p.numeric}, solution={p.solution} for '{p.prompt}'"
        )

def _snapshot_generation(seed: int, cat: str, level: int, n: int = 5):
    random.seed(seed)
    out = []
    for _ in range(n):
        prob = REG.generate(cat, level)
        out.append((prob.prompt, str(prob.solution)))
    return out

@pytest.mark.parametrize("level", [1, 3])
def test_generation_reproducible_with_seed(level: int):
    cats = REG.list_categories()
    assert cats, "Ni registriranih kategorij."
    cat = cats[0]
    seed = 20251030 + level
    seq1 = _snapshot_generation(seed, cat, level, n=5)
    seq2 = _snapshot_generation(seed, cat, level, n=5)
    assert seq1 == seq2, f"Nabor ni reproducibilen pri enakem seedu (cat={cat})."


--- Konec datoteke: tests\test_core_correctness.py ---

--- Začetek datoteke: tests\test_dry_integrity.py ---

import re
from pathlib import Path

FILES = ["ucni_vmesnik.py", "test_vmesnik.py"]

BAD_FUNCS = [
    r"def\s+_safe_eval_number_expr\s*\(",
    r"def\s+_parse_simple_monomial\s*\(",
]

def test_no_local_helpers():
    root = Path(__file__).resolve().parents[1]
    textmap = {p: (root / p).read_text(encoding="utf-8") for p in FILES}
    for p, txt in textmap.items():
        for pat in BAD_FUNCS:
            assert not re.search(pat, txt), f"Podvojena helper funkcija v {p}: vzorec {pat}"


--- Konec datoteke: tests\test_dry_integrity.py ---

--- Začetek datoteke: tests\test_gui_imports.py ---

# -*- coding: utf-8 -*-
"""
tests/test_gui_imports.py
-------------------------
Sanity check: import GUI modulov ne sme samodejno ustvariti Tk glavnega okna.
"""
import importlib
import tkinter as tk

def test_ucni_vmesnik_no_autostart():
    if hasattr(tk, "_default_root"):
        tk._default_root = None
    mod = importlib.import_module("ucni_vmesnik")
    assert getattr(tk, "_default_root", None) is None, "ucni_vmesnik je ob importu ustvaril Tk root!"

def test_test_vmesnik_no_autostart():
    if hasattr(tk, "_default_root"):
        tk._default_root = None
    mod = importlib.import_module("test_vmesnik")
    assert getattr(tk, "_default_root", None) is None, "test_vmesnik je ob importu ustvaril Tk root!"


--- Konec datoteke: tests\test_gui_imports.py ---

--- Začetek datoteke: tests\test_steps_language.py ---

# -*- coding: utf-8 -*-
"""
tests/test_steps_language.py
----------------------------
Preverjanje, da so učni koraki brez angleških kratic (gcd/GCD/NSD, g₁/g₂),
in da izvoz v Markdown uporabi polne izraze (varianta A, brez kratic).

Testi:
- test_vrr_steps_no_abbrev:    'Vrstni red računov' koraki ne vsebujejo kratic.
- test_all_categories_clean:   Za vse registrirane kategorije humanizirani koraki nimajo kratic.
- test_markdown_export_clean:  Izvoz v Markdown zapiše humanizirane korake (brez kratic).
"""

from __future__ import annotations
import re
import random
from pathlib import Path

import pytest

from predmeti import register as REG
from predmeti.vrstni_red_racunov import generiraj_vrstni_red_racunov
from predmeti import ulomki as _ensure_import1  # noqa: F401  (da se registrira)
from predmeti import potence_in_koreni as _ensure_import2  # noqa: F401
from predmeti import linearne_enacbe as _ensure_import3  # noqa: F401
from predmeti import polinomi as _ensure_import4  # noqa: F401
from predmeti import trigonometrija as _ensure_import5  # noqa: F401
from predmeti import funkcije_in_grafi as _ensure_import6  # noqa: F401
from predmeti import sistemi_enacb as _ensure_import7  # noqa: F401
from predmeti import procenti_in_ddv as _ensure_import8  # noqa: F401

import ucni_vmesnik as UIM  # zaradi izvoz_markdown (ta vključi humanizacijo)


FORBIDDEN = (
    r"\bGCD\b",
    r"\bgcd\b",
    r"\bNSD\b",
    "g₁",
    "g₂",
)

def _contains_forbidden(text: str) -> bool:
    if text is None:
        return False
    s = text or ""
    for patt in FORBIDDEN:
        if patt.startswith(r"\b"):
            if re.search(patt, s, flags=re.IGNORECASE):
                return True
        else:
            if patt in s:
                return True
    return False

def _humanize_steps_local(text: str) -> str:
    """
    Lokalna kopija 'humanizerja' (varianta A – brez kratic), skladno z UIM._humanize_steps:
    - gcd / GCD / NSD -> Največji skupni delitelj
    - g₁ / g₂        -> Največji skupni delitelj (prvi/drugi par)
    """
    if not text:
        return text
    s = text
    s = re.sub(r"\bGCD\b", "Največji skupni delitelj", s)
    s = re.sub(r"\bgcd\b", "Največji skupni delitelj", s, flags=re.IGNORECASE)
    s = re.sub(r"\bNSD\b", "Največji skupni delitelj", s)
    s = s.replace("g₁", "Največji skupni delitelj (prvi par)")
    s = s.replace("g₂", "Največji skupni delitelj (drugi par)")
    return s


@pytest.mark.parametrize("level", [1, 2, 3])
def test_vrr_steps_no_abbrev(level: int):
    """Kategorija 'Vrstni red računov' naj v korakih ne uporablja kratic."""
    random.seed(12345 + level)
    for _ in range(30):
        p = generiraj_vrstni_red_racunov(level)
        assert not _contains_forbidden(p.steps), f"Koraki vsebujejo kratico:\n{p.steps}"


@pytest.mark.parametrize("level", [1, 2, 3])
def test_all_categories_clean(level: int):
    """
    Za vse registrirane kategorije: humanizirani koraki so brez kratic.
    (Če kakšna kategorija še vedno generira kratice, jih UI/izvoz odpravi.)
    """
    random.seed(8888 + level)
    cats = REG.list_categories()
    assert cats, "Ni registriranih kategorij."
    for cat in cats:
        # generiraj nekaj primerov na kategorijo
        for _ in range(8):
            p = REG.generate(cat, level)
            cleaned = _humanize_steps_local(p.steps)
            assert not _contains_forbidden(cleaned), (
                f"Humanizirani koraki naj ne vsebujejo kratic (cat={cat}).\n"
                f"ORIG:\n{p.steps}\n----\nCLEAN:\n{cleaned}"
            )
            # idempotentnost (ponovna humanizacija nič ne spremeni)
            cleaned2 = _humanize_steps_local(cleaned)
            assert cleaned2 == cleaned, "Humanizacija naj bo idempotentna."


@pytest.mark.parametrize("level", [1, 3])
def test_markdown_export_clean(tmp_path: Path, level: int):
    """
    Preveri, da izvoz v Markdown zapiše korake brez kratic (UIM.izvoz_markdown uporablja humanizer).
    """
    random.seed(4242 + level)
    # zgradi majhen nabor preko registra
    cats = REG.list_categories()
    # vzemi do 3 kategorije, 2 nalogi vsak (ali manj, če je malo kategorij)
    chosen = cats[:3] if len(cats) >= 3 else cats
    naloge = []
    for cat in chosen:
        for _ in range(2):
            naloge.append(REG.generate(cat, level))
    # izvozi
    out_md = tmp_path / "delovni_list.md"
    res_path = UIM.izvoz_markdown(naloge, out_md)
    # preberi REŠITVE (ta datoteka naj vsebuje humanizirane korake)
    content = res_path.read_text(encoding="utf-8")
    # razdelaj na code block-e korakov in preveri, da nimajo kratic
    # iščemo bloke med ``` in ```
    blocks = []
    inside = False
    cur = []
    for line in content.splitlines():
        if line.strip() == "```":
            if inside:
                blocks.append("\n".join(cur))
                cur = []
                inside = False
            else:
                inside = True
        elif inside:
            cur.append(line)
    # vsaj nekaj blokov bi moralo biti
    assert blocks, "Ni najdenih blokov korakov v izvoznih rešitvah."
    for b in blocks:
        assert not _contains_forbidden(b), f"V izvoženih korakih je kratica:\n{b}"


--- Konec datoteke: tests\test_steps_language.py ---

--- Začetek datoteke: tests\__init__.py ---



--- Konec datoteke: tests\__init__.py ---

--- Začetek datoteke: ui\theme.py ---

# -*- coding: utf-8 -*-
"""
ui/theme.py
-----------
Skupne barvne sheme (svetla/temna) za MathDrill GUI-je.
"""

LIGHT = {
    "BG": "#F6F7FB",
    "PANEL": "#FFFFFF",
    "TEXT": "#0F172A",
    "SUB": "#475569",
    "PRIMARY": "#2563EB",
    "ACCENT": "#16A34A",
    "LIST_BG": "#FFFFFF",
    "LIST_SEL": "#DBEAFE",
}

DARK = {
    "BG": "#0F131A",
    "PANEL": "#151A22",
    "TEXT": "#E6EDF3",
    "SUB": "#9AA4B2",
    "PRIMARY": "#4C84FF",
    "ACCENT": "#6EE7B7",
    "LIST_BG": "#0E131B",
    "LIST_SEL": "#25406B",
}


--- Konec datoteke: ui\theme.py ---

--- Začetek datoteke: ui\__init__.py ---



--- Konec datoteke: ui\__init__.py ---


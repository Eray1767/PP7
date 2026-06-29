# PP7

## Goal

In this exercise you will:

* **Understand** each stage of the C compilation pipeline (preprocessing, compilation to assembly, assembling, and linking).
* **Inspect** intermediate outputs to see how your C code transforms at each step.
* **Use** text‑processing tools (`grep`, `sed`, `awk`) with regular expressions to search and replace patterns in code.
* **Automate** search/replace interactively and non‑interactively in **Vim**.
* **Modularize** C code by defining functions across separate files and **link** them together manually.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. When time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository on GitHub.
2. **Clone** your fork to your local machine.
3. Create a directory at the repository root named `solutions/`:

   ```bash
   mkdir solutions
   ```
4. For each task below, add the required files under `solutions/` (for example, `solutions/sample.c`, `solutions/sample.i`, etc.).
5. **Commit** your changes locally, then **push** them to GitHub.
6. **Submit** your repository link for review.

---

## Prerequisites

* A C compiler toolchain (e.g. `gcc`).
* Familiarity with command‑line tools: `grep`, `sed`, `awk`, and `vim`.
* Man‑pages for reference:

  ```bash
  man gcc   # for compiler options
  man grep  # for searching
  man sed   # for stream editing
  man awk   # for pattern scanning
  man vim   # for interactive editing
  ```

---

## Tasks

### Task 1: C Compilation Stages

**Objective:** Break down the process of transforming C source into an executable and inspect each intermediate file.

1. In `solutions/`, create a C source file named `sample.c` containing a simple `main()`:

   ```c
   #include <stdio.h>

   int main(void) {
       printf("Hello, PP7!\n");
       return 0;
   }
   ```
2. **Preprocess** only (run the preprocessor, `-E`) and save the output:

   ```bash
   gcc -E sample.c -o solutions/sample.i
   ```

   * Open `solutions/sample.i` in an editor and note how `#include <stdio.h>` expands and macros are handled.
3. **Compile to assembly** (`-S`):

   ```bash
   gcc -S solutions/sample.i -o solutions/sample.s
   ```

   * Examine `solutions/sample.s` to see the generated assembly instructions for `printf` and `return`.
4. **Assemble** (`-c`):

   ```bash
   gcc -c solutions/sample.s -o solutions/sample.o
   ```

   * Verify that `solutions/sample.o` is an object file (e.g., with `file sample.o`).
5. **Link** to produce an executable:

   ```bash
   gcc solutions/sample.o -o solutions/sample
   ```

   * Run `./solutions/sample` and confirm it prints `Hello, PP7!`.
6. **Explain** in comments or a short README how each stage transforms the code.

1. Preprocessing (gcc -E)
  Der Präprozessor führt alle Anweisungen aus, die mit # beginnen.

  Was passiert hier?

  #include <stdio.h> wird durch den kompletten Inhalt der Header‑Datei ersetzt

  Makros werden aufgelöst

  Kommentare werden entfernt

 Ergebnis:  
Eine große Textdatei (sample.i), die reinen C‑Code enthält, aber ohne Includes und Makros.

2. Compilation zu Assembly (gcc -S)
Der Compiler übersetzt den präprozessierten C‑Code in Assembly‑Code.

Was passiert hier?

C‑Code wird analysiert

Optimierungen werden angewendet

Der Code wird in CPU‑spezifische Assembly‑Befehle übersetzt

Ergebnis:  
Eine .s‑Datei mit Assembly‑Instruktionen (z. B. mov, call, ret).

3. Assembling (gcc -c)
Der Assembler wandelt den Assembly‑Code in Maschinencode um.

Was passiert hier?

Assembly‑Befehle werden in Binärcode übersetzt

Relocation‑Informationen werden erzeugt

Ergebnis:  
Eine Objektdatei (sample.o), die aber noch nicht ausführbar ist.

4. Linking (ohne Flag, nur gcc sample.o -o sample)
Der Linker verbindet alle Objektdateien und Bibliotheken zu einem fertigen Programm.

Was passiert hier?

sample.o wird mit der Standardbibliothek (z. B. libc) verknüpft

Externe Symbole wie printf werden aufgelöst

Ein ausführbares Programm entsteht
---

### Task 2: Regex Search & Replace in Code

**Objective:** Use `grep`, `sed`, and `awk` to locate and modify code patterns, then perform the same edits in **Vim** both interactively and via the CLI.

1. Copy `solutions/sample.c` to `solutions/debug_sample.c`:

   ```bash
   cp solutions/sample.c solutions/debug_sample.c
   ```
2. **Search** for all calls to `printf` using `grep` with a regex pattern and save the results:

   ```bash
   grep -En "printf\s*\(" solutions/debug_sample.c
   ```
3. **Replace** each `printf` with `debug_printf` in the file using `sed` (in‑place):

   ```bash
   sed -E -i.bak 's/printf\s*/debug_printf/g' solutions/debug_sample.c
   ```
4. **Extract** lines containing `debug_printf` using `awk`:

   ```bash
   awk '/debug_printf/ { print NR, $0 }' solutions/debug_sample.c
   ```
5. **Vim Interactive**: open `solutions/debug_sample.c` in Vim:

   ```bash
   vim solutions/debug_sample.c
   ```

   * Use search (`/printf`) and substitute (`:%s/printf/debug_printf/g`) commands interactively.
   * Save and quit.
6. **Vim CLI**: perform the same substitute without opening the full UI:

   ```bash
   vim -c ":%s/printf/debug_printf/g" -c ":wq" solutions/debug_sample.c
   ```
7. **Explain** each tool’s approach to regex-based search and replace, and when you might prefer one over the others.
grep durchsucht Dateien nach Mustern und zeigt passende Zeilen an.
Es eignet sich gut, um schnell herauszufinden, wo bestimmte Funktionen vorkommen.

  sed führt Ersetzungen direkt im Text durch (stream editing).
  Ideal für automatisches, schnelles Suchen/Ersetzen in Dateien.

  awk ist ein Muster- und Textprozessor, der Zeilen analysiert und formatierte Ausgaben erzeugt.
  Perfekt, wenn man Zeilennummern oder bestimmte Felder ausgeben möchte.

  Vim interaktiv erlaubt manuelles Suchen/Ersetzen und ist gut für kontrollierte Änderungen.
  Vim im CLI-Modus automatisiert dieselben Änderungen ohne Benutzeroberfläche.
---

### Task 3: Modular Linking with `extern`

**Objective:** Create a separate addition function in `add.c`, declare it `extern` in `main.c`, and compile/link manually to form an executable.

1. In `solutions/`, create `add.c`:

   ```c
   // add.c
   int add(int a, int b) {
       return a + b;
   }
   ```
2. Create `main.c` that uses `add`:

   ```c
   // main.c
   #include <stdio.h>
   extern int add(int, int);

   int main(void) {
       int sum = add(5, 7);
       printf("5 + 7 = %d\n", sum);
       return 0;
   }
   ```
3. **Compile** each source separately:

   ```bash
   gcc -c solutions/add.c -o solutions/add.o
   gcc -c solutions/main.c -o solutions/main.o
   ```
4. **Link** the object files manually:

   ```bash
   gcc solutions/add.o solutions/main.o -o solutions/add_example
   ```
5. Run `./solutions/add_example` to verify it prints `5 + 7 = 12`.
6. **Document** the workflow in comments or a short file, emphasizing:

   * The role of `extern` declarations.
   * Why separating compilation can speed up builds.
   * How manual linking differs from letting `gcc` handle all steps in one command.
Beide Dateien wurden separat kompiliert: gcc -c add.c -o add.o gcc -c main.c -o main.o

Danach wurden die Objektdateien manuell gelinkt: gcc add.o main.o -o add_example

Beim Ausführen zeigt das Programm das Ergebnis der Addition.
---

**Remember:** Stop working after **90 minutes** and record where you stopped.

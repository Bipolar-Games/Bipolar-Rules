# Struktura folderów

```
Assets/
├── _Project/            ← WSZYSTKIE pliki projektu
│   ├── Scenes/          ← każda scena we własnym folderze o tej samej nazwie (Level01/Level01.unity)
│   ├── Scripts/         ← podfoldery wg systemów (Camera/, Actors/, ...)
│   └── Settings/        ← assety URP, profile Volume
├── Bipolar/             ← moduły studia (każdy z własnym README.md)
└── <ZewnętrznaBiblioteka>/  ← paczki z Asset Store / zewnętrzne biblioteki
```

Zasady:

- **Nasze pliki trafiają wyłącznie do `Assets/_Project/`.** Nowe skrypty, sceny, prefaby, materiały, tekstury, audio — wszystko tam, w odpowiednim podfolderze.
- **Bezpośrednio w `Assets/` leżą tylko zewnętrzne biblioteki** (Asset Store, pluginy, importowane paczki) **oraz moduły studia w `Assets/Bipolar/`**. Nie twórz tam plików projektu.
- **Każda scena ma własny folder o tej samej nazwie co scena**, w `Assets/_Project/Scenes/` (np. `Scenes/Level01/Level01.unity`). Pozostałe pliki należące do sceny trzymamy w jej folderze.
- **Nie modyfikuj zewnętrznych bibliotek ani modułów studia.** Jeśli trzeba zmienić ich zachowanie — rozszerz je (wrapper, dziedziczenie, adapter) w `Assets/_Project/`.
- Namespace skryptu odpowiada jego folderowi pod `Scripts/` (namespace bazowy: patrz `SKILL.md`).

---
name: bipolar-rules
description: Wspólne zasady studia Bipolar Games dla projektów Unity - styl kodu C# (hermetyzacja, SOLID, eventy, kolejność składowych klasy, formatowanie if/for, NaughtyAttributes, Input System) oraz struktura folderów Assets/_Project i używanie modułów studia z Assets/Bipolar (serializacja interfejsów, interakcje, sceny) zamiast pisania własnych odpowiedników. Używaj przy pisaniu, zmienianiu i refaktoryzacji skryptów C# (MonoBehaviour, ScriptableObject), dodawaniu nowych plików do projektu Unity oraz zawsze, gdy użytkownik zadaje pytanie zamiast wydawać polecenie.
---

# BIPOLAR_RULES — wspólne zasady studia Bipolar Games

Uniwersalne zasady komunikacji, struktury projektu i stylu kodu, wspólne dla wszystkich projektów Unity studia Bipolar.
Instrukcje specyficzne dla projektu są w `AGENTS.md` (lub odpowiedniku) danego repozytorium.

> **Nie edytuj tego pliku w trakcie pracy w repozytorium.** To kopia wspólnego szablonu dodawanego
> do każdego projektu. Zmiany wprowadzamy w szablonie i kopiujemy do projektów. Jeśli zasada dotyczy
> tylko jednego projektu, wpisz ją do `AGENTS.md` tego projektu.
> W razie konfliktu pierwszeństwo mają zasady z `AGENTS.md` projektu.

## Pytanie ≠ polecenie

- **Gdy użytkownik zadaje pytanie, odpowiedz na nie i nic nie zmieniaj.** Nie edytuj plików, nie commituj, nie wprowadzaj zmian „przy okazji”. Czytać kod wolno, żeby odpowiedzieć.
- **Działaj dopiero na wyraźne polecenie** („zrób”, „dodaj”, „popraw”, „zacommituj” itp.).
- Jeśli w odpowiedzi proponujesz zmianę, opisz ją i poczekaj na decyzję.

## Struktura folderów

```
Assets/
├── _Project/            ← WSZYSTKIE pliki projektu
│   ├── Scenes/
│   ├── Scripts/         ← podfoldery wg systemów (Camera/, Actors/, ...)
│   └── Settings/        ← assety URP, profile Volume
├── Bipolar/             ← moduły studia (każdy z własnym README.md)
└── <ZewnętrznaBiblioteka>/  ← paczki z Asset Store / zewnętrzne biblioteki
```

Zasady:

- **Nasze pliki trafiają wyłącznie do `Assets/_Project/`.** Nowe skrypty, sceny, prefaby, materiały, tekstury, audio — wszystko tam, w odpowiednim podfolderze.
- **Bezpośrednio w `Assets/` leżą tylko zewnętrzne biblioteki** (Asset Store, pluginy, importowane paczki) **oraz moduły studia w `Assets/Bipolar/`**. Nie twórz tam plików projektu.
- **Nie modyfikuj zewnętrznych bibliotek.** Jeśli trzeba zmienić ich zachowanie — rozszerz je (wrapper, dziedziczenie, adapter) w `Assets/_Project/`.
- Namespace odpowiada folderowi pod `Scripts/` (patrz „Styl kodu C#”).

## Moduły studia (`Assets/Bipolar/`)

Studio ma własne, gotowe moduły (np. serializacja interfejsów, system interakcji, zarządzanie scenami). Zestaw modułów różni się między projektami, więc ich opisy **nie** są w tym skillu. Każdy moduł ma własny `README.md`.

- **Zanim napiszesz nowy system** (serializacja, interakcje, sceny, zarządzanie zasobami itp.), **sprawdź, czy w `Assets/Bipolar/` jest moduł do tego zadania.** Przejrzyj foldery modułów i przeczytaj `README.md` każdego pasującego.
- **Jeśli moduł pasuje, użyj go. Nie pisz własnego odpowiednika.**
- Jeśli żaden moduł nie pasuje albo nie masz pewności, zapytaj, zanim zaczniesz pisać własne rozwiązanie.
- Nie modyfikuj modułów w repozytorium projektu (zasada jak dla zewnętrznych bibliotek). Brakującą funkcję zaproponuj lub rozszerz w `Assets/_Project/`.
- Wzór README modułu: `module-readme-template.md`.

## Styl kodu C#

- **Namespace bazowy** to nazwa projektu (np. `MagicJam`); namespace odpowiada folderowi pod `Scripts/`.

### Hermetyzacja (najważniejsze)

Stosujemy zasady **SOLID**, a przede wszystkim **hermetyzację**:

- **Pola są zawsze prywatne.** Żadnych publicznych pól.
- **Na zewnątrz wystawiamy tylko właściwości** (najlepiej tylko do odczytu) i metody.
- Pola widoczne w Inspectorze to `private` + `[SerializeField]`, nigdy `public`.
- Setter publiczny tylko wtedy, gdy naprawdę jest potrzebny; preferuj `private set` lub metodę opisującą intencję (`Enable()`, `ApplyDamage(...)`).
- Nie wystawiaj mutowalnych kolekcji — zwracaj `IReadOnlyList<T>` / `IReadOnlyCollection<T>`.

### Atrybuty — w linijce POWYŻEJ pola

`[SerializeField]` (i pozostałe atrybuty: `[Header]`, `[Tooltip]`, `[Range]` itd.) piszemy **w osobnej linijce nad polem**, nie w tej samej linii.

```csharp
public class OperatorCamera : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField]
    private float moveSpeed = 3f;

    [Tooltip("Time to reach target velocity. 0 = instant.")]
    [SerializeField]
    private float moveSmoothTime = 0.12f;

    private Camera cam;

    public Camera Camera => cam;
    public float MoveSpeed => moveSpeed;
    public bool InputEnabled { get; private set; } = true;
}
```

Źle:

```csharp
public float moveSpeed = 3f;                    // publiczne pole
[SerializeField] private float moveSpeed = 3f;  // atrybut w tej samej linii
```

### Pozostałe zasady SOLID w praktyce

- **Single Responsibility:** jeden `MonoBehaviour` = jedna odpowiedzialność. Zamiast jednego wielkiego managera — kilka małych komponentów.
- **Open/Closed, Dependency Inversion:** zależności przez interfejsy lub referencje ustawiane w Inspectorze (`[SerializeField]`), nie przez `FindObjectOfType` / `GameObject.Find` w runtime (w `Reset()` wolno, patrz `reset-references.md`).
- **Interface Segregation:** małe, wąskie interfejsy (`IInteractive`, a nie `IEverything`).
- Komunikacja między systemami przez zdarzenia (`event Action<...>`) zamiast bezpośrednich wywołań w obie strony i cyklicznych zależności.

### Referencje podpinane w `Reset()`

Referencje do komponentów i obiektów sceny podpinamy automatycznie w `Reset()` (`GetComponent`, `...InChildren`, `...InParent`, a menedżery ze sceny przez `FindFirstObjectByType`). Szczegóły i przykłady: `reset-references.md`.

### Eventy

Domyślnie C#-owe eventy (`Action`, własny delegat od dwóch argumentów, nazwy z prefiksem `On`), `UnityEvent` tylko dla efektów niewpływających na grę. Szczegóły i przykłady: `events.md`.

### Enumy i typy rozszerzalne

- **`enum` tylko dla zamkniętych, niezmiennych zbiorów** (kierunki, proste stany). Typy NPC, postaci, przedmiotów i inne rodzaje, które mogą się rozrastać, to `ScriptableObject`, nie enum.
- Szczegóły, wyjątki i przykłady: `enums-and-extensible-types.md` — **czytaj przy każdej propozycji dodania enuma lub typu/rodzaju czegokolwiek.**

### Konwencje nazewnicze

- Klasy, metody, właściwości, eventy: `PascalCase`.
- Pola prywatne i zmienne lokalne: `camelCase` (bez prefiksu `_` / `m_`).
- Stałe: `PascalCase` (`private const float Skin = 0.01f;`).
- Interfejsy z prefiksem `I`.
- Eventy z prefiksem `On` (`OnOpened`, `OnCreating`), handlery `Obiekt_NazwaEventu` (`Door_OnOpened`). Publiczne metody nie zaczynają się od `On` (szczegóły w `events.md`).
- **`MonoBehaviour` i `ScriptableObject` — zawsze we własnym pliku, nazwa pliku = nazwa klasy** (wymóg Unity).
- Pozostałe typy publiczne (zwykłe klasy, struktury, enumy, delegaty) mogą dzielić plik, gdy są ze sobą ściśle powiązane.
- Komentarze i nazwy w kodzie po angielsku; `<summary>` dla publicznych klas i nieoczywistych członków.

### Input

Pakiet `com.unity.inputsystem` nazywamy „Input System”, nie „nowy Input System”. Jeśli go używamy, komponenty przyjmują akcje przez `[SerializeField] InputActionReference` (zgodnie z regułą atrybutu powyżej pola).

## Co jest w którym pliku

Pliki leżą obok `SKILL.md`. Czytaj je wtedy, gdy dotyczą zadania:

| Plik | Kiedy czytać |
|---|---|
| `formatting.md` | przy każdym pisaniu lub zmianie kodu C# (klamry, `else`, negacja, obiekty Unity) |
| `class-layout.md` | przy każdym pisaniu lub zmianie kodu C# (kolejność składowych, stany podglądane w Inspectorze) |
| `events.md` | przy eventach, delegatach i `UnityEvent` |
| `reset-references.md` | przy podpinaniu referencji w `Reset()` |
| `naughty-attributes.md` | przy atrybutach Inspectora z NaughtyAttributes |
| `enums-and-extensible-types.md` | przy każdym enumie lub typie/rodzaju czegokolwiek |
| `module-readme-template.md` | przy tworzeniu README modułu w `Assets/Bipolar/` |

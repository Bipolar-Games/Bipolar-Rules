---
name: bipolar-rules
description: Wspólne zasady studia Bipolar Games dla projektów Unity - styl kodu C# (hermetyzacja, SOLID, formatowanie, kolejność składowych klasy, nazewnictwo, eventy, enumy, Reset, RequireComponent, NaughtyAttributes, Input System), struktura folderów Assets/_Project i moduły studia z Assets/Bipolar. Używaj przy każdym pisaniu, zmienianiu i refaktoryzacji skryptów C# (MonoBehaviour, ScriptableObject) oraz przy dodawaniu plików i scen do projektu Unity.
---

# BIPOLAR_RULES — wspólne zasady studia Bipolar Games

Uniwersalne zasady struktury projektu i stylu kodu, wspólne dla wszystkich projektów Unity studia Bipolar.
Instrukcje specyficzne dla projektu są w `AGENTS.md` (lub odpowiedniku) danego repozytorium.

> **Nie edytuj tego pliku w trakcie pracy w repozytorium.** To kopia wspólnego szablonu dodawanego
> do każdego projektu. Zmiany wprowadzamy w szablonie i kopiujemy do projektów. Jeśli zasada dotyczy
> tylko jednego projektu, wpisz ją do `AGENTS.md` tego projektu.
> W razie konfliktu pierwszeństwo mają zasady z `AGENTS.md` projektu.

## Co przeczytać przed zadaniem

Pliki leżą obok `SKILL.md`. **Przed rozpoczęciem zadania przeczytaj wszystkie pliki z wiersza, który pasuje do tego, co robisz** (jedno zadanie może pasować do kilku wierszy).

| Gdy… | Przeczytaj |
|---|---|
| piszesz lub zmieniasz **jakikolwiek kod C#** | `formatting.md`, `class-layout.md`, `naming.md` |
| tworzysz lub zmieniasz **komponent** (`MonoBehaviour`, `ScriptableObject`), dodajesz mu pola lub referencje | powyższe + `component-references.md` |
| komponent potrzebuje **referencji** do innego komponentu lub obiektu sceny (`RequireComponent`, `Reset()`, `GetComponent`, `Find...`) | `component-references.md` |
| dodajesz **event, delegat** lub subskrypcję, nazywasz handler | `events.md` |
| dodajesz **enum** albo typ/rodzaj czegokolwiek (NPC, przedmiot, stan, kategoria) | `enums-and-extensible-types.md` |
| obsługujesz **wejście gracza** (Input System, `InputActionReference`) | `input.md` |
| używasz atrybutów Inspectora z NaughtyAttributes (`Button`, `ShowNativeProperty` itd.) lub pokazujesz stan w Inspectorze | `naughty-attributes.md` |
| dodajesz **nowy plik, scenę, prefab, asset** lub nie wiesz, gdzie je położyć | `project-structure.md` |
| piszesz **nowy system** (serializacja, interakcje, sceny, zasoby) albo potrzebujesz referencji do interfejsu w Inspectorze | `studio-modules.md` |
| tworzysz README modułu w `Assets/Bipolar/` | `module-readme-template.md` |

## Zasady stosowane zawsze

- **Namespace bazowy** to nazwa projektu (np. `MagicJam`); namespace odpowiada folderowi pod `Scripts/`.
- Stosujemy zasady **SOLID**, a przede wszystkim **hermetyzację**.

### Hermetyzacja

- **Pola są zawsze prywatne.** Żadnych publicznych pól.
- **Na zewnątrz wystawiamy tylko właściwości** (najlepiej tylko do odczytu) i metody.
- Pola widoczne w Inspectorze to `private` + `[SerializeField]`, nigdy `public`.
- Setter publiczny tylko wtedy, gdy naprawdę jest potrzebny; preferuj `private set` lub metodę opisującą intencję (`Enable()`, `ApplyDamage(...)`).
- Nie wystawiaj mutowalnych kolekcji — zwracaj `IReadOnlyList<T>` / `IReadOnlyCollection<T>`.

### Atrybuty — w linijce POWYŻEJ pola

`[SerializeField]` (i pozostałe atrybuty: `[Header]`, `[Tooltip]`, `[Range]` itd.) piszemy **w osobnej linijce nad polem**, nie w tej samej linii.

**Wyjątek: atrybut na argumencie funkcji piszemy w tej samej linii** co argument (`void Log([CallerMemberName] string caller = null)`).

```csharp
public class OperatorCamera : MonoBehaviour
{
    [Header("Movement")]
    [SerializeField]
    private float moveSpeed = 3f;
    public float MoveSpeed => moveSpeed;

    [Tooltip("Time to reach target velocity. 0 = instant.")]
    [SerializeField]
    private float moveSmoothTime = 0.12f;

    private Camera cam;
    public Camera Camera => cam;

    public bool InputEnabled { get; private set; } = true;
}
```

Źle:

```csharp
public float moveSpeed = 3f;                    // publiczne pole
[SerializeField] private float moveSpeed = 3f;  // atrybut w tej samej linii
```

Poprawnie (wyjątek dla argumentów):

```csharp
private void Log(string message, [CallerMemberName] string caller = null) { ... }
```

### Pozostałe zasady SOLID w praktyce

- **Single Responsibility:** jeden `MonoBehaviour` = jedna odpowiedzialność. Zamiast jednego wielkiego managera — kilka małych komponentów.
- **Open/Closed, Dependency Inversion:** zależności przez interfejsy lub referencje ustawiane w Inspectorze (`[SerializeField]`), nie przez `FindFirstObjectByType` / `FindAnyObjectByType` / `GameObject.Find` w runtime. Wyjątek: `Reset()` (patrz `component-references.md`).
- **Interface Segregation:** małe, wąskie interfejsy (`IInteractive`, a nie `IEverything`).
- Komunikacja między systemami przez zdarzenia (`event Action<...>`) zamiast bezpośrednich wywołań w obie strony i cyklicznych zależności (patrz `events.md`).

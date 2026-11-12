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
- **Open/Closed, Dependency Inversion:** zależności przez interfejsy lub referencje ustawiane w Inspectorze (`[SerializeField]`), nie przez `FindObjectOfType` / `GameObject.Find` w runtime (w `Reset()` wolno, patrz „Referencje podpinane w `Reset()`”).
- **Interface Segregation:** małe, wąskie interfejsy (`IInteractive`, a nie `IEverything`).
- Komunikacja między systemami przez zdarzenia (`event Action<...>`) zamiast bezpośrednich wywołań w obie strony i cyklicznych zależności.

### Referencje podpinane w `Reset()`

Gdy komponent potrzebuje referencji do innych komponentów, podpinamy je automatycznie w `Reset()`. Unity wywołuje `Reset()` tylko w edytorze: przy dodaniu komponentu do obiektu oraz po wybraniu *Reset* z menu komponentu. Wynik zapisuje się w polu `[SerializeField]`, więc w grze nie ma kosztu wyszukiwania.

- **Komponent może być na tym samym obiekcie** → `GetComponent<T>()`.
- **Komponent może być w dzieciach** → `GetComponentInChildren<T>()`, **w rodzicu** → `GetComponentInParent<T>()`.
- **Komponent wymaga referencji do obiektu na scenie** (np. menedżera) → w `Reset()` wolno użyć wyszukiwania po całej scenie: `FindObjectOfType<T>()` lub `FindFirstObjectByType<T>()` (zależnie od wersji Unity). To wyjątek od zakazu `Find...` w runtime, bo `Reset()` nie działa w grze.
- Pola, które wypełnia `Reset()`, to nadal `private` + `[SerializeField]`. Można je poprawić ręcznie w Inspectorze.
- Wyszukiwanie po scenie w `Reset()` działa na obiektach z otwartej sceny lub prefaba. Referencji do obiektów z innej sceny w ten sposób nie zapiszesz.
- `Reset()` jest pierwszą metodą w kolejności składowych (przed `Awake`).

```csharp
public class DoorSoundBridge : MonoBehaviour
{
    [SerializeField]
    private Door door;

    [SerializeField]
    private AudioSource audioSource;

    [SerializeField]
    private Room room;

    [SerializeField]
    private LevelManager levelManager;

    private void Reset()
    {
        door = GetComponent<Door>();
        audioSource = GetComponentInChildren<AudioSource>();
        room = GetComponentInParent<Room>();
        levelManager = FindFirstObjectByType<LevelManager>();
    }
}
```

Źle:

```csharp
private void Awake()
{
    levelManager = FindFirstObjectByType<LevelManager>();   // wyszukiwanie po scenie w runtime; powinno być w Reset()
}
```

### Eventy

- **Event bez argumentów lub z jednym argumentem — zwykły `Action` / `Action<T>`.**
- **Od dwóch argumentów w górę — własny delegat**, żeby każdy argument miał nazwę.
- Delegat nie potrzebuje osobnego pliku. Jeśli event z tym delegatem wywołuje jedna klasa, deklarujemy delegat w jej pliku.

```csharp
public event Action Retaken;
public event Action<FilmMode> ModeChanged;
```

```csharp
// Health.cs — delegat w pliku jedynej klasy, która wywołuje ten event
public delegate void DamageTakenHandler(float amount, Vector3 hitPoint);

public class Health : MonoBehaviour
{
    public event DamageTakenHandler DamageTaken;
}
```

Źle:

```csharp
public event Action<float, Vector3> DamageTaken;  // co oznacza float, a co Vector3?
```

#### C# eventy zamiast `UnityEvent`

- **Domyślnie używamy C#-owych eventów (`event Action` / własny delegat), nie `UnityEvent`.** Są spójne i widoczne w kodzie.
- **Wszystko, od czego zależy mechanika, progresja lub w ogóle działanie gry, idzie przez C#-owe eventy.** Bez wyjątków.
- **`UnityEvent` jest dopuszczalny tylko dla efektów, które nie wpływają na działanie gry** (dźwięk, wizualia), i tylko bardzo rzadko.
- **Nawet dla efektów preferuj osobny komponent-mostek:** jeden komponent kontroluje mechanikę i wystawia event, drugi odpowiada za efekt, a mostek nasłuchuje eventu mechaniki i wywołuje efekt. Mechanika nie wie nic o efektach.
- Nie dodawaj `UnityEvent` sam z siebie. Jeśli uważasz, że ma sens, zaproponuj to i poczekaj na decyzję.

```csharp
// Door.cs — mechanika, nic nie wie o dźwięku
public class Door : MonoBehaviour
{
    public event Action Opened;

    public void Open()
    {
        // logika otwarcia
        Opened?.Invoke();
    }
}

// DoorSoundBridge.cs — mostek: nasłuchuje mechaniki, odpala efekt
public class DoorSoundBridge : MonoBehaviour
{
    [SerializeField]
    private Door door;

    [SerializeField]
    private AudioSource openSound;

    private void OnEnable()
    {
        door.Opened += OnOpened;
    }

    private void OnDisable()
    {
        door.Opened -= OnOpened;
    }

    private void OnOpened()
    {
        openSound.Play();
    }
}
```

Źle:

```csharp
public class Door : MonoBehaviour
{
    [SerializeField]
    private UnityEvent onOpened;   // UnityEvent wołający logikę gry (np. odblokowanie poziomu) przez Inspector

    public void Open()
    {
        onOpened.Invoke();
    }
}
```

### Enumy i typy rozszerzalne

- **`enum` tylko dla zamkniętych, niezmiennych zbiorów** (kierunki, proste stany). Typy NPC, postaci, przedmiotów i inne rodzaje, które mogą się rozrastać, to `ScriptableObject`, nie enum.
- Szczegóły, wyjątki i przykłady: `enums-and-extensible-types.md` — **czytaj przy każdej propozycji dodania enuma lub typu/rodzaju czegokolwiek.**

### Konwencje nazewnicze

- Klasy, metody, właściwości, eventy: `PascalCase`.
- Pola prywatne i zmienne lokalne: `camelCase` (bez prefiksu `_` / `m_`).
- Stałe: `PascalCase` (`private const float Skin = 0.01f;`).
- Interfejsy z prefiksem `I`.
- **`MonoBehaviour` i `ScriptableObject` — zawsze we własnym pliku, nazwa pliku = nazwa klasy** (wymóg Unity).
- Pozostałe typy publiczne (zwykłe klasy, struktury, enumy, delegaty) mogą dzielić plik, gdy są ze sobą ściśle powiązane.
- Komentarze i nazwy w kodzie po angielsku; `<summary>` dla publicznych klas i nieoczywistych członków.

### Formatowanie instrukcji

- **Instrukcja po `if` / `for` / `foreach` / `while` zawsze w osobnej linijce** — nigdy w tej samej linii co warunek.
- **Jednolinijkowe ciało `if` / `for` / `foreach` / `while` piszemy bez klamer.** Klamry tylko dla ciał wieloliniowych.
- **Zasady dla `if` / `for` / `foreach` / `while` są wspólne.** Wszystkie te konstrukcje (warunki i pętle) traktujemy tak samo.
- **Zagnieżdżone konstrukcje (`if` / `for` / `foreach` / `while` w innym `if` / `for` / `foreach` / `while`) traktujemy jako jedną całość — klamry są albo wszędzie, albo nigdzie:**
  - Jeśli **którakolwiek** konstrukcja w łańcuchu musi mieć klamry (bo ma ciało wieloliniowe), **wszystkie** konstrukcje w tym łańcuchu, zewnętrzne i wewnętrzne, mają klamry, nawet te jednolinijkowe.
  - Jeśli **żadna** nie musi mieć klamer (każda ma jednolinijkowe ciało), **żadna ich nie ma**.
  - **Wyjątek: warunek, którego ciałem jest wyłącznie `return`, `break` lub `continue`, nigdy nie dostaje klamer** (zawsze w osobnej linijce), także wewnątrz konstrukcji z klamrami.
- **Łańcuch `if` / `else if` / `else` to jedna całość:** jeśli którakolwiek gałąź musi mieć klamry, wszystkie gałęzie je mają.
- **Unikamy `else` i `else if`.** Zazwyczaj się da:
  - przez wydzielenie logiki do osobnej metody, która w pierwszej gałęzi kończy się `return` po `if`,
  - przez operator ternarny przy prostym przypisaniu wartości.
- **Negację w warunkach zapisujemy jako `== false`, nie przez `!`.**
- **Obiekty Unity (`UnityEngine.Object`: komponenty, `GameObject`, assety):**
  - czy **istnieje** → `if (target)`,
  - czy **nie istnieje** → `if (target == null)` (nie `if (!target)` ani `if (target == false)`).

```csharp
if (isActive == false)
    return;

for (int i = 0; i < actors.Count; i++)
    actors[i].Reset();

if (target == null)
    return;

if (target && target.IsReady == false)
{
    target.Prepare();
    target.Notify();
}

// zagnieżdżone, wszystkie jednolinijkowe: bez klamer nigdzie
for (int i = 0; i < actors.Count; i++)
    if (actors[i].IsDead)
        actors[i].Remove();

// zagnieżdżone, wewnętrzna wieloliniowa: klamry wszędzie
for (int i = 0; i < actors.Count; i++)
{
    if (actors[i].IsDead)
    {
        actors[i].Remove();
        count++;
    }
}

// wyjątek: strażnik z continue / break / return bez klamer, mimo klamer na zewnątrz
for (int i = 0; i < actors.Count; i++)
{
    if (actors[i] == null)
        continue;

    if (actors[i].IsDead)
    {
        actors[i].Remove();
        count++;
    }
}

// zamiast else: osobna metoda z return po if
private string GetLabel()
{
    if (isLocked)
        return "Locked";

    return "Open";
}

// zamiast if / else przy przypisaniu: operator ternarny
string label = isLocked ? "Locked" : "Open";

// else nieunikniony, jedna gałąź ma klamry: wszystkie gałęzie mają klamry
if (isLocked)
{
    ShowLock();
    PlaySound();
}
else
{
    Open();
}
```

Źle:

```csharp
if (!isActive) return;                        // instrukcja w tej samej linii, negacja przez !
for (int i = 0; i < n; i++) { Do(i); }        // zbędne klamry i instrukcja w tej samej linii
if (target != null)                           // istnienie obiektu Unity sprawdzamy przez if (target)
    target.Prepare();
if (!target)                                  // brak obiektu Unity sprawdzamy przez target == null
    return;

for (int i = 0; i < actors.Count; i++)
{
    if (actors[i].IsDead)                     // wewnętrzna jednolinijkowa bez klamer, gdy zewnętrzna je ma
        actors[i].Remove();
    Tick(actors[i]);
}

for (int i = 0; i < actors.Count; i++)
{
    if (actors[i] == null)
    {                                         // klamry przy samym continue / break / return
        continue;
    }
}

if (isLocked)
{
    ShowLock();
    PlaySound();
}
else
    Open();                                   // jedna gałąź ma klamry, druga nie

if (isLocked)
    label = "Locked";
else                                          // zbędny else: ternary albo osobna metoda z return
    label = "Open";
```

### Kolejność składowych klasy

- **Eventy (`public event ...`) deklarujemy na samej górze klasy**, przed polami, właściwościami i metodami.
- Dalej, w tej kolejności: pola serializowane, pozostałe pola prywatne, właściwości, metody.
- **Metody układamy według kolejności wywołań w silniku Unity:**
  1. `Reset` (wyjątek: przed `Awake`),
  2. `Awake`,
  3. `OnEnable`,
  4. `Start`,
  5. **środek klasy, w dowolnej kolejności:** metody publiczne, prywatne metody pomocnicze, `Update`, `FixedUpdate`, `LateUpdate` i pozostałe komunikaty Unity (`OnTriggerEnter`, `OnCollisionEnter` itd.),
  6. sprzątanie: `OnDisable`, potem `OnDestroy`,
  7. na samym dole metody edytorowe: `OnValidate`, `OnDrawGizmos`, `OnDrawGizmosSelected`.

```csharp
public class Door : MonoBehaviour
{
    private void Reset() { ... }
    private void Awake() { ... }
    private void OnEnable() { ... }
    private void Start() { ... }

    public void Open() { ... }
    private void Update() { ... }
    private void OnTriggerEnter(Collider other) { ... }
    private bool CanOpen() { ... }

    private void OnDisable() { ... }
    private void OnDestroy() { ... }

    private void OnValidate() { ... }
    private void OnDrawGizmos() { ... }
}
```
- **Stany podglądane w Inspectorze** to pola i właściwości, które zmieniają się w trakcie gry i chcemy je widzieć do debugowania (np. `isMoving`, `shotTime`).
  - Pola stanu oznaczamy `[SerializeField]`, choć ich zapis w scenie nie jest potrzebny. **Nie używamy `[NaughtyAttributes.ShowNonSerializedField]`**, bo źle się wyświetla.
  - Właściwości stanu oznaczamy `[NaughtyAttributes.ShowNativeProperty]` z NaughtyAttributes (patrz „NaughtyAttributes”).
  - Grupujemy je w jednym bloku po pozostałych polach prywatnych, z nagłówkiem `[Header("States")]` nad pierwszym polem. `[Header]` działa tylko na polach, więc gdy blok zaczyna się od właściwości, użyj komentarza `// States`.

```csharp
public class ModeController : MonoBehaviour
{
    /// <summary>Raised after the mode changed; the argument is the new mode.</summary>
    public event Action<FilmMode> ModeChanged;

    [SerializeField]
    private InputActionReference toggleAction;

    [Header("States")]
    [SerializeField]
    private int lastTickSecond;

#if NAUGHTY_ATTRIBUTES
    [NaughtyAttributes.ShowNativeProperty]
#endif
    public FilmMode Mode { get; private set; }

    private void Awake() { ... }
}
```

### NaughtyAttributes

Systemy mają działać **zarówno z biblioteką NaughtyAttributes, jak i bez niej**. Dlatego:

- **Każdy atrybut z NaughtyAttributes otaczamy `#if NAUGHTY_ATTRIBUTES` … `#endif`.**
- **Namespace podajemy inline przy atrybucie** (`[NaughtyAttributes.Button]`), **nie** przez `using NaughtyAttributes;` nad klasą. `using` poza `#if` nie skompiluje się bez biblioteki.
- Kod nie może zależeć od NaughtyAttributes w działaniu: bez biblioteki znikają tylko udogodnienia w Inspectorze.
- Symbol `NAUGHTY_ATTRIBUTES` musi być zdefiniowany w projekcie, który używa biblioteki (np. *Version Defines* w `.asmdef` albo *Scripting Define Symbols* w Player Settings).

```csharp
public class ActorController : MonoBehaviour
{
#if NAUGHTY_ATTRIBUTES
    [NaughtyAttributes.ShowNativeProperty]
#endif
    public ActorState State { get; private set; }

#if NAUGHTY_ATTRIBUTES
    [NaughtyAttributes.Button]
#endif
    private void Retake() { ... }
}
```

Źle:

```csharp
using NaughtyAttributes;          // nie skompiluje się bez biblioteki

public class ActorController : MonoBehaviour
{
    [ShowNativeProperty]          // brak #if NAUGHTY_ATTRIBUTES
    public ActorState State { get; private set; }
}
```

### Input

Pakiet `com.unity.inputsystem` nazywamy „Input System”, nie „nowy Input System”. Jeśli go używamy, komponenty przyjmują akcje przez `[SerializeField] InputActionReference` (zgodnie z regułą atrybutu powyżej pola).
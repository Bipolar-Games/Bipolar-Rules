# Referencje do komponentów i obiektów sceny

Jak komponent zdobywa referencje do innych komponentów i obiektów sceny. Wybór sposobu zależy od tego, czy referencja jest wymagana, opcjonalna, czy dotyczy obiektu sceny.

## Komponent wymagany na tym samym obiekcie — `RequireComponent`

- **Jeśli klasa wymaga komponentu na tym samym obiekcie, ma nad deklaracją klasy `[RequireComponent(typeof(T))]`**, w osobnej linijce.
- **Pola wymaganego komponentu nie serializujemy.** Komponent na pewno jest na obiekcie, więc sposób jego pobrania jest oczywisty i nie podpinamy go w Inspectorze ani w `Reset()`. Pobieramy go w `Awake()` albo przez właściwość, która przy pierwszym odczycie zapisuje referencję w polu.

```csharp
[RequireComponent(typeof(Door))]
public class DoorSoundBridge : MonoBehaviour
{
    private Door door;

    private void Awake()
    {
        door = GetComponent<Door>();
    }
}
```

```csharp
[RequireComponent(typeof(Door))]
public class DoorSoundBridge : MonoBehaviour
{
    private Door door;

    private Door Door
    {
        get
        {
            if (door == null)
                door = GetComponent<Door>();

            return door;
        }
    }
}
```

Źle:

```csharp
public class DoorSoundBridge : MonoBehaviour      // brak [RequireComponent(typeof(Door))]
{
    private Door door;

    private void Awake()
    {
        door = GetComponent<Door>();
    }
}

[RequireComponent(typeof(Door))]
public class DoorSoundBridge : MonoBehaviour
{
    [SerializeField]                               // wymagany komponent nie jest serializowany ani podpinany w Reset()
    private Door door;

    private void Reset()
    {
        door = GetComponent<Door>();
    }
}
```

## Komponent opcjonalny lub z innego obiektu — `Reset()`

**Komponent opcjonalny albo taki, który może być na innym obiekcie, nie ma `RequireComponent`.** Podpinamy go automatycznie w `Reset()`. Unity wywołuje `Reset()` tylko w edytorze: przy dodaniu komponentu do obiektu oraz po wybraniu *Reset* z menu komponentu. Wynik zapisuje się w polu `[SerializeField]`, więc w grze nie ma kosztu wyszukiwania.

- **W `Reset()` używamy wyłącznie** `GetComponentInChildren<T>()` (przeszukuje też własny obiekt), `GetComponentInParent<T>()` oraz wyszukiwania po scenie (`Find...`, patrz niżej).
- **Komponent wymaga referencji do obiektu na scenie** (np. menedżera) → w `Reset()` wolno użyć wyszukiwania po całej scenie (metody: patrz „Wyszukiwanie obiektów po typie”). To wyjątek od zakazu `Find...` w runtime, bo `Reset()` nie działa w grze.
- Pola, które wypełnia `Reset()`, to nadal `private` + `[SerializeField]`. Można je poprawić ręcznie w Inspectorze.
- Wyszukiwanie po scenie w `Reset()` działa na obiektach z otwartej sceny lub prefaba. Referencji do obiektów z innej sceny w ten sposób nie zapiszesz.
- Miejsce `Reset()` w klasie: `class-layout.md`.

```csharp
public class DoorSoundBridge : MonoBehaviour
{
    [SerializeField]
    private AudioSource audioSource;

    [SerializeField]
    private Room room;

    [SerializeField]
    private LevelManager levelManager;

    private void Reset()
    {
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

## Wyszukiwanie obiektów po typie (Unity 6)

W Unity 6 `FindObjectOfType` jest oznaczone jako przestarzałe (dokumentacja: „Use Object.FindFirstObjectByType instead or if finding any instance is acceptable the faster Object.FindAnyObjectByType”). Używamy nowych metod:

| Potrzeba | Używaj | **Nie używaj** |
|---|---|---|
| jeden obiekt, wynik ma być przewidywalny | `FindFirstObjectByType<T>()` | `FindObjectOfType<T>()` |
| jeden obiekt, wystarczy dowolny (np. singletonowy menedżer) | `FindAnyObjectByType<T>()` (szybsze, niegwarantowana kolejność) | `FindObjectOfType<T>()` |
| wiele obiektów | `FindObjectsByType<T>(FindObjectsSortMode.None)` | `FindObjectsOfType<T>()` |

- Domyślnie metody pomijają nieaktywne obiekty. Nieaktywne włączamy parametrem `FindObjectsInactive.Include`, nie starym `bool includeInactive`.
- `FindObjectsByType` wymaga `FindObjectsSortMode`. Gdy kolejność nie ma znaczenia, podajemy `None` (szybsze).
- Wyszukiwanie po scenie jest kosztowne. Wolno go używać w `Reset()`, nigdy w `Update` ani w innych często wywoływanych metodach (zasada ogólna: `SKILL.md`).

```csharp
levelManager = FindFirstObjectByType<LevelManager>();
var doors = FindObjectsByType<Door>(FindObjectsSortMode.None);
```

Źle:

```csharp
levelManager = FindObjectOfType<LevelManager>();   // przestarzałe w Unity 6
var doors = FindObjectsOfType<Door>();             // przestarzałe w Unity 6
```

## Runtime — `TryGetComponent`

**Komponent, którego brak jest dopuszczalny** (na tym samym lub innym obiekcie), pobieramy w runtime przez `TryGetComponent` i obsługujemy jego brak.

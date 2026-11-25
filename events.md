# Eventy

- **Event bez argumentów lub z jednym argumentem — zwykły `Action` / `Action<T>`.**
- **Od dwóch argumentów w górę — własny delegat**, żeby każdy argument miał nazwę.
- Delegat nie potrzebuje osobnego pliku. Jeśli event z tym delegatem wywołuje jedna klasa, deklarujemy delegat w jej pliku.
- **Nazwa eventu zaczyna się od `On`** (`OnOpened`, `OnModeChanged`). Dzięki temu po wpisaniu `On` IDE pokazuje listę eventów obiektu.
- **Czas gramatyczny rozróżnia moment:** czas przeszły to zdarzenie po fakcie (`OnCreated`), czas teraźniejszy ciągły to zdarzenie przed nim (`OnCreating`).
- **Do eventu subskrybujemy bezpośrednio metodę, która wykonuje pracę**, jeśli przyjmuje dokładnie te same argumenty, które wychodzą z eventu (`gameManager.OnStateChanged += RefreshView;`). Nie tworzymy dodatkowej metody-opakowania tylko po to, żeby miała nazwę `Obiekt_NazwaEventu`.
- **Wzorzec Visual Studio `Obiekt_NazwaEventu` (`GameManager_OnStateChanged`) stosujemy tylko i wyłącznie wtedy, gdy metody wykonującej pracę nie da się zasubskrybować, bo ma inną sygnaturę niż event.** Metoda `Obiekt_NazwaEventu` przyjmuje wtedy argumenty eventu i wywołuje właściwą metodę.
- **Opakowanie z jednym wywołaniem i zgodną sygnaturą jest zbędne:** subskrybujemy wtedy wywoływaną metodę bezpośrednio.
- **Publiczne metody nie zaczynają się od `On`.** Prefiks `On` jest zarezerwowany dla eventów i komunikatów Unity (`OnEnable`, `OnTriggerEnter` itd.).

```csharp
public event Action OnRetaken;
public event Action<FilmMode> OnModeChanged;
```

```csharp
// Health.cs — delegat w pliku jedynej klasy, która wywołuje ten event
public delegate void DamageTakenHandler(float amount, Vector3 hitPoint);

public class Health : MonoBehaviour
{
    public event DamageTakenHandler OnDamageTaken;
}
```

Źle:

```csharp
public event Action<float, Vector3> OnDamageTaken;  // co oznacza float, a co Vector3?
```

## C# eventy zamiast `UnityEvent`

- **Domyślnie używamy C#-owych eventów (`event Action` / własny delegat), nie `UnityEvent`.** Są spójne i widoczne w kodzie.
- **Wszystko, od czego zależy mechanika, progresja lub w ogóle działanie gry, idzie przez C#-owe eventy.** Bez wyjątków.
- **`UnityEvent` jest dopuszczalny tylko dla efektów, które nie wpływają na działanie gry** (dźwięk, wizualia), i tylko bardzo rzadko.
- **Nawet dla efektów preferuj osobny komponent-mostek:** jeden komponent kontroluje mechanikę i wystawia event, drugi odpowiada za efekt, a mostek nasłuchuje eventu mechaniki i wywołuje efekt. Mechanika nie wie nic o efektach.
- Nie dodawaj `UnityEvent` sam z siebie. Jeśli uważasz, że ma sens, zaproponuj to i poczekaj na decyzję.

```csharp
// Door.cs — mechanika, nic nie wie o dźwięku
public class Door : MonoBehaviour
{
    public event Action OnOpened;

    public void Open()
    {
        // logika otwarcia
        OnOpened?.Invoke();
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
        door.OnOpened += PlayOpenSound;
    }

    private void OnDisable()
    {
        door.OnOpened -= PlayOpenSound;
    }

    private void PlayOpenSound()
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

## Nazwy metod subskrybowanych do eventów

Sygnatura metody zgadza się z eventem: subskrybujemy ją bezpośrednio.

```csharp
public event Action OnStateChanged;       // GameManager.cs

private void RefreshView() { ... }

gameManager.OnStateChanged += RefreshView;
```

Sygnatura się różni: dopiero wtedy dodajemy metodę we wzorcu Visual Studio.

```csharp
public event Action<GameState> OnStateChanged;   // GameManager.cs

private void RefreshView() { ... }                // nie przyjmuje GameState

gameManager.OnStateChanged += GameManager_OnStateChanged;

private void GameManager_OnStateChanged(GameState state)
{
    RefreshView();
}
```

Źle:

```csharp
gameManager.OnStateChanged += GameManager_OnStateChanged;   // Action bez argumentów, RefreshView pasuje wprost

private void GameManager_OnStateChanged()                   // zbędne opakowanie tylko z nazwą wzorca
{
    RefreshView();
}
```

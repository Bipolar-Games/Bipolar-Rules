# Eventy

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

# Kolejność składowych klasy

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
  - Właściwości stanu oznaczamy `[NaughtyAttributes.ShowNativeProperty]` z NaughtyAttributes (patrz `naughty-attributes.md`).
  - Grupujemy je w jednym bloku po pozostałych polach prywatnych, z nagłówkiem `[Header("States")]` nad pierwszym polem. `[Header]` działa tylko na polach, więc gdy blok zaczyna się od właściwości, użyj komentarza `// States`.

```csharp
public class ModeController : MonoBehaviour
{
    /// <summary>Raised after the mode changed; the argument is the new mode.</summary>
    public event Action<FilmMode> OnModeChanged;

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

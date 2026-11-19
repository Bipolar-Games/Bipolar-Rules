# Input

Pakiet `com.unity.inputsystem` nazywamy „Input System”, nie „nowy Input System”. Jeśli go używamy, komponenty przyjmują akcje przez `[SerializeField] InputActionReference` (atrybut w linijce nad polem).

Właściciel akcji jest oddzielony od jej konsumentów:

- **Akcje (mapy akcji) włącza i wyłącza jeden właściciel**, np. `PlayerInput` albo własny komponent wejścia, zależnie od stanu gry (rozgrywka, UI itd.).
- **Konsumenci tylko subskrybują** `performed` / `canceled` w `OnEnable` i odsubskrybowują w `OnDisable`. Odsubskrybowanie wyłącza akcję dla tego komponentu i nie wpływa na pozostałych.
- **Konsument nie wywołuje `Enable()` ani `Disable()` na akcji współdzielonej**, bo stan włączenia nie jest zliczany: `Disable()` w jednym komponencie wyłącza akcję dla wszystkich.

```csharp
public class Jumper : MonoBehaviour
{
    [SerializeField]
    private InputActionReference jumpAction;

    private void OnEnable()
    {
        jumpAction.action.performed += JumpAction_Performed;
    }

    private void OnDisable()
    {
        jumpAction.action.performed -= JumpAction_Performed;
    }

    private void JumpAction_Performed(InputAction.CallbackContext context)
    {
        Jump();
    }
}
```

Źle:

```csharp
private void OnEnable()
{
    jumpAction.action.Enable();                 // konsument włącza współdzieloną akcję
    jumpAction.action.performed += JumpAction_Performed;
}

private void OnDisable()
{
    jumpAction.action.Disable();                // wyłączy akcję także innym komponentom
    jumpAction.action.performed -= JumpAction_Performed;
}
```

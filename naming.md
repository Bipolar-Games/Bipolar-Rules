# Nazewnictwo

## Konwencje

- Klasy, metody, właściwości, eventy: `PascalCase`.
- Pola prywatne i zmienne lokalne: `camelCase` (bez prefiksu `_` / `m_`).
- Stałe: `PascalCase` (`private const float Skin = 0.01f;`).
- Interfejsy z prefiksem `I`.
- Eventy, handlery eventów i metody z prefiksem `On`: zasady w `events.md`.
- Korutyny: patrz niżej.
- Typy publiczne (zwykłe klasy, struktury, enumy, delegaty) mogą dzielić plik, gdy są ze sobą ściśle powiązane.
- **Słowa amerykańskie, nigdy brytyjskie** (`Color`, `Behavior`, `Center`, `Initialize`). Wyjątek: `MonoBehaviour` i inne nazwy z Unity lub bibliotek, których nie da się zmienić.
- Komentarze i nazwy w kodzie po angielsku. `<summary>` dla publicznych klas i nieoczywistych członków jest opcjonalne: nie trzeba go pisać.

## Korutyny

Korutyna to specjalny rodzaj metody, więc ma własne zasady nazewnictwa:

- **Nazwa korutyny nie jest czasownikiem.** Wywołanie korutyny w Unity tworzy obiekt, który ją wykonuje, więc nazwa opisuje tę czynność jak rzeczownik, najczęściej odczasownikowy.
- **Nazwa kończy się przyrostkiem `Co`**, żeby od razu było widać, czym jest, i że nie wywołujemy jej jak zwykłej metody (tylko przez `StartCoroutine`).
- **Część przed `Co` opisuje czynność**, którą korutyna wykonuje.

```csharp
private IEnumerator ScreenFadeCo() { ... }
private IEnumerator ElementsCountingCo() { ... }

StartCoroutine(ScreenFadeCo());
```

Źle:

```csharp
private IEnumerator FadeScreen() { ... }   // wygląda jak zwykła metoda, brak przyrostka Co
FadeScreen();                              // nie uruchomi korutyny
```

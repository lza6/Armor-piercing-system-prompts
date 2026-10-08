# Polish: patterns that make a draft sound generated

Read this when the draft is in Polish. It lists wording that machine-written and
machine-translated Polish overuses, with plainer alternatives, and the checks that
matter in Polish specifically. Facts, numbers, commands, and quotations follow the
same rules as in `SKILL.md`; nothing here permits changing them.

## Stock openers and fillers

Use these as context-dependent editing suggestions, not banned phrases. Keep wording
that carries emphasis, uncertainty, scope, or an intentional register. Replace a
phrase only when the replacement preserves the same claim.

| Overused | Usually better |
| --- | --- |
| Z ogromną / wielką przyjemnością informujemy, że… | Say what happened: „Dodaliśmy…”, „Od dziś…” |
| Mamy zaszczyt ogłosić… | same |
| Warto zauważyć, że… / Należy podkreślić, że… | Delete; state the point |
| W dzisiejszych czasach… / W dzisiejszym dynamicznym świecie… | Delete |
| Nie da się ukryć, że… / Nie ulega wątpliwości, że… | Remove only when it adds no meaningful certainty or emphasis |
| W kontekście… / W ramach… (when it adds nothing) | „w”, „przy”, „podczas” |
| kluczowy, innowacyjny, przełomowy, kompleksowy, zaawansowany | Name the concrete property, or delete |
| umożliwia / pozwala na (as a reflex) | Prefer „pozwala podejrzeć”; do not turn a capability into an action already performed |
| w celu + noun | „aby” + verb, or „żeby” |
| dokonać + noun (dokonać instalacji, dokonać zmiany) | The verb: „zainstalować”, „zmienić” |
| posiadać (for things) | „mieć” |
| szereg, wiele różnych, cały wachlarz | Keep the quantity; use a specific number only if supplied by the source |
| Podsumowując, … (at the end of a short text) | Delete |
| Mam nadzieję, że ten artykuł okazał się pomocny. | Delete |

## Calques from English

Machine translation leaves English structure in place. Typical signs:

- **Passive where Polish uses an impersonal or active form.** „Plik został
  zapisany przez program” → „Program zapisał plik” or „Plik zapisano”.
- **„jest w stanie”** for every *can* → „może”, when this preserves capability; do not turn ability into completion.
- **Overused „że”-clauses after reporting verbs.** „Testy pokazują, że wynik jest
  lepszy” is fine; repeated clauses may need restructuring for readability, but are not evidence of AI authorship.
- **Capitalized common nouns** copied from English titles or UI: „Ustawienia
  Użytkownika” → „ustawienia użytkownika” unless it is the exact label of a button.
- **Sentence-initial „Także,” or „Dodatkowo,”** as connectors → join the sentences or
  drop the connector.
- **„Wspierać” for *support* in the sense of *handle*** → „obsługiwać” („obsługuje
  Windows”), keep „wspierać” for backing a cause or a person.
- **„Aplikować”** for *apply* → „stosować”, „zastosować”; „aplikować” is for job
  applications.
- **„Adresować problem”** → „zająć się problemem”; use „rozwiązać” only if the source promises a solution.
- **„Domyślnie” placed like English *by default*** at sentence end → usually earlier:
  „Domyślnie tryb jest wyłączony”.
- **Decimal points and thousands separators.** Polish prose uses a decimal comma
  („2,5 MB”) but code, versions, and flags keep their exact form (`2.5`, `v1.2.0`).
  Do not convert numbers inside code spans or commands.

## Register and address

- Choose one form of address and keep it: „Ty” (lowercase in running text unless the
  project capitalizes it), „Państwo”, or impersonal („można”, „należy”). Mixing them
  in one paragraph reads as generated.
- Avoid inventing a gender for the author or reader. Impersonal forms („zrobiono”,
  „można zapisać”) can avoid guessing gender. Keep the source narrator and number: do not introduce
  „my” or a team by changing a singular statement to „sprawdziliśmy”.
  If the source already uses a gendered past tense, keep it.
- Diminutives („pliczek”, „opcyjka”) and forced jokes are not what makes a text
  human. Concrete verbs and normal sentence length do.
- Keep English technical terms the project already uses („commit”, „pull request”,
  „deploy”) and inflect them the way the project does. Do not translate a term the
  glossary keeps in English.

## Checks that are specific to Polish

- **Negation words to keep:** nie, nigdy, bez, żaden/żadna/żadne, ani, nikt, nic.
  A dropped „nie” reverses the claim. Double negation („nie ma nic”) is correct
  Polish and must stay.
- **Hedges to keep:** może, mogą, prawdopodobnie, chyba, około, wstępnie, tylko,
  jeszcze, na razie, w tym teście. „Nie sprawdziliśmy jeszcze” and „nie sprawdziliśmy”
  are different claims.
- **Aspect carries meaning.** „Zapisywał” (was saving) and „zapisał” (saved) differ;
  do not switch aspect to make a sentence shorter.
- **Diacritics must survive** in prose (ż, ó, ł, ą, ę, ś, ć, ń, ź) and must not be
  added to identifiers, flags, or code.
- **Quotation marks:** Polish prose uses „…” ; keep the source's marks inside
  quotations and code exactly.
- `scripts/check_facts.py` counts a fixed subset of Polish negation and hedge words,
  including „nie” and „jeszcze”. Phrases such as „na razie” and „w tym teście”
  still need manual comparison. It does not check aspect, case, or diacritics.

## Worked examples

**Release note**

Source: „Z ogromną przyjemnością informujemy, że w ramach naszych ciągłych starań o
najwyższą jakość dokonaliśmy implementacji innowacyjnego mechanizmu, który umożliwia
użytkownikom podgląd zmian przed ich zapisaniem. Warto zauważyć, że funkcja ta jest
obecnie w fazie testów i może nie działać na systemie Windows.”

One suitable edit: „Można teraz podejrzeć zmiany przed ich zapisaniem. Funkcja jest w
fazie testów i może nie działać w Windows.”

Acceptance: the preview behavior, the testing status, and „może nie działać” survive.
No claim about quality, effort, or users is added. `check_facts.py` reports no
dropped negation or hedge.

**Maintainer reply**

Source: „Dziękujemy za zgłoszenie! Nie da się ukryć, że jest to bardzo istotny
problem. Aby móc dokonać jego analizy, uprzejmie prosimy o dostarczenie informacji
dotyczących wersji systemu operacyjnego oraz wersji Pythona, a także pełnego
polecenia, które zostało wykonane.”

One suitable edit: „Dzięki za zgłoszenie. To istotny problem. Żeby go sprawdzić,
potrzebujemy wersji systemu i Pythona oraz pełnego uruchomionego polecenia.”

Acceptance: all three requested items remain; the reply does not claim the bug was
reproduced. `check_facts.py` reports one dropped „nie”: it belonged to the filler „Nie
da się ukryć”. In this edit the assessment „istotny problem” remains, and removing
the rhetorical opener is an intentional reduction of emphasis. Explain that choice
rather than silently reverting or declaring that the phrase never carries meaning.
The reply does not guess the reporter's gender.

These are authored examples, not recorded client evaluations. A fluent reader should
review edits before they are published.

---
project: go-hotel
version: 1
status: draft
created: 2026-06-17
context_type: greenfield
product_type: web-app
target_scale:
  users: small
  qps: low
  data_volume: small
timeline_budget:
  mvp_weeks: null
  hard_deadline: null
  after_hours_only: true
---

## Vision & Problem Statement

Istniejące gry hotelowe na mobile są przeładowane reklamami i nie dają czystej przyjemności z rozgrywki. Gracz casual nie ma dobrej alternatywy na krótkie sesje (2–5 min) bez przerywania co chwilę reklamami.

Go-hotel to prosta gra mobilna/webowa bez reklam, w której jesteś właścicielem hotelu — budujesz go od zera, zatrudniasz pracowników, automatyzujesz operacje i rozwijasz go w czasie. Projekt osobisty z ambicją publicznego releasu.

**Typ bólu:** brakująca jakość — istniejące gry w gatunku są złe z powodu nadmiaru reklam.

**Insight:** gracz, który sam buduje grę z AI, wie od środka co psuje doświadczenie — i może tego uniknąć od pierwszego dnia.

**Gatunek:** hybryd idle + tycoon — hotel działa pasywnie między sesjami, ale w dłuższych sesjach gracz aktywnie zarządza i podejmuje decyzje.

## User & Persona

**Persona główna:** Casual gracz mobilny — ty sam oraz osoby o podobnym profilu.

- Wiek: 20–40 lat
- Kontekst użycia: krótkie sesje 2–5 min (toaleta, kolejka, przerwa w pracy)
- Oczekiwania: szybki progress widoczny w jednej sesji; brak reklam; gra "czeka" na gracza gdy wróci
- Motywacja: satysfakcja z budowania i rozwijania czegoś własnego
- Frustracja z istniejących gier: agresywna monetyzacja, przerywające reklamy

## Success Criteria

### Primary
MVP flow (dowód że gra działa):
1. Gracz otwiera aplikację — od razu widok hotelu 2D, bez menu głównego
2. Recepcja sygnalizuje czekającego gościa (pulsująca ikona)
3. Gracz klika "przyjmij gościa" → postać idzie do recepcji (animacja)
4. Gość przypisany do pokoju, idzie sam
5. Po kilku sekundach pokój jest "brudny"
6. Gracz klika "sprzątaj pokój X" → postać idzie, animuje sprzątanie
7. Pokój gotowy → kolejny gość może wejść
8. Gracz widzi narastający przychód i może kupić odblokowanie drugiego pokoju

### Secondary
- Drugi pokój do odblokowania za zarobione pieniądze — pierwsza pętla progresji: zarabiasz → kupujesz → więcej zarabiasz

### Guardrails
- Postępy gracza przeżywają odświeżenie przeglądarki i zamknięcie aplikacji — stan gry jest trwały między sesjami
- Zero reklam w jakiejkolwiek formie

## User Stories

### US-01: Cykl obsługi gościa

- **Given** hotel jest otwarty, jest co najmniej jeden wolny, czysty pokój
- **When** gracz klika na pulsującą recepcję i akceptuje gościa
- **Then** gość idzie do pokoju, po odejściu pokój jest oznaczony jako brudny, gracz może go posprzątać, po sprzątaniu pokój wraca do stanu "wolny" i przychód gracza rośnie

#### Acceptance Criteria
- Gość nie trafia do pokoju zajętego — przydział jest deterministyczny
- Stan pokoju (wolny / zajęty / brudny) jest widoczny dla gracza w każdym momencie
- Przychód po zakończonym cyklu zwiększa licznik kasy
- Cykl można powtórzyć bez restartu gry

## Functional Requirements

### Rdzeń rozgrywki
- FR-001: Gracz może przyjąć gościa przy recepcji. Priority: must-have
  > Socrates: Ryzyko zidentyfikowane: jeśli kliknięcie jest wymagane na każdego gościa, frustruje w idle mode. Rozwiązanie: w v1 aktywne, ale system musi obsługiwać opcjonalny tryb "auto-przyjmij" w v2.
- FR-002: Gracz może skierować postać do sprzątania pokoju. Priority: must-have
  > Socrates: Ręczna faza jest celowa — satysfakcja z pracy ręcznej, zanim automatyzacja ją zastąpi. Wzorzec idle: "robiłem to sam → teraz pracownik robi za mnie" daje wartość automatyzacji.
- FR-003: Gość automatycznie idzie do przypisanego pokoju. Priority: must-have
  > Socrates: Ryzyko: błąd przydziału (gość idzie do zajętego pokoju) niszczy iluzję kontroli. Rozwiązanie: logika przydziału pokoi musi być deterministyczna i odporna na edge cases.

### Ekonomia i progresja
- FR-004: System zapisuje i przywraca stan gry (postępy, pieniądze, pokoje). Priority: must-have
  > Socrates: Brak kontr-argumentu. Zapis to absolutna podstawa.
- FR-005: Gracz widzi stan kasy z paskiem postępu do kolejnego odblokowania (np. "340 / 500 do Pokój 2"). Priority: must-have
  > Socrates: Ryzyko zidentyfikowane: sama liczba bez kontekstu celu nie daje satysfakcji. Rozwiązanie: widoczny cel "ile do odblokowania" jest częścią core loop.
- FR-006: Gracz może odblokować drugi pokój za zarobione pieniądze. Priority: must-have
  > Socrates: Jeden odblok wystarczy żeby pokazać pętlę. Czas do "gra skończona" zależy od tempa zarabiania — wymaga kalibracji.

### Animacje i automatyzacja
- FR-007: Postać gracza animuje się podczas wykonywania zadań. Priority: must-have
  > Socrates: Awans z nice-to-have — bez animacji to arkusz kalkulacyjny, nie gra.
- FR-008: Gracz może zatrudnić pierwszego pracownika do automatycznego sprzątania. Priority: must-have
  > Socrates: Awans z nice-to-have — bez pracownika gracz nie widzi dokąd zmierza. Core fantasy hotelu (delegowanie → automatyzacja) musi być widoczne w MVP.

## Non-Functional Requirements

- Czas uruchomienia gry < 3 sekundy na średnio szybkim urządzeniu mobilnym (mierzony jako czas do pierwszej interaktywnej klatki)
- Podstawowa rozgrywka działa bez połączenia z internetem; stan gry jest synchronizowany gdy połączenie jest dostępne
- Gra jest grywalna w przeglądarce mobilnej; architektura umożliwia publikację w natywnych sklepach mobilnych (Play Store, App Store) bez przepisywania kodu — wymaganie dla v1.1, nice-to-have na MVP
- Zero reklam — żadna forma reklamy w interfejsie gry

## Business Logic

Hotel generuje przychód za każdego obsłużonego gościa. Liczba wolnych, czystych pokoi to jedyna zmienna która skaluje przychód — gracz maksymalizuje ją przez szybkie sprzątanie lub delegowanie do pracownika. Odblokowanie kolejnych pokoi wymaga osiągnięcia progu gotówki, tworząc pętlę: obsłuż gości → zarabiaj → odblokuj → obsłuż więcej gości.

Gra nie ocenia jakości obsługi (brak systemu ocen gości w MVP) — liczy się wyłącznie przepustowość: ile pokoi działa, jak szybko wracają do stanu "wolny".

## Access Control

- Brak wymaganego logowania — gra startuje natychmiast, zapis lokalny domyślnie
- Opcjonalne logowanie przez konto zewnętrzne (social sign-in) odblokowuje synchronizację zapisu między urządzeniami — feature v2, nie MVP
- Model ról: płaski — jeden typ gracza, każdy ma swój własny hotel

## Non-Goals

- Brak trybu multiplayer ani rankingów — każdy gracz ma swój prywatny hotel, zero interakcji między graczami
- Brak jakiejkolwiek monetyzacji (IAP, subscription, reklamy) — MVP bezpłatne; decyzja po walidacji gry
- Brak eventów sezonowych / specjalnych — bez świątecznych content drops ani time-limited challenges
- Brak systemu ocen gości — gość płaci i wychodzi, zero gwiazdek ani recenzji w MVP

## Open Questions

1. **Kalibracja ekonomiki gry** — ile zarabia gracz za gościa? Jaki jest koszt odblokowania drugiego pokoju? Jaki jest koszt pracownika? Wartości definiują tempo core loop i muszą być dobrane eksperymentalnie po implementacji. Owner: user. Block: nie — iteracja po MVP.
2. **Endgame i docelowa liczba pokoi** — MVP ma 2 pokoje. Ile łącznie pokoi planowane jest w finalnej wizji gry? Odpowiedź kształtuje architekturę systemu pokoi. Owner: user. Block: nie — MVP nie wymaga tej odpowiedzi.
3. **`mvp_weeks` = null** — użytkownik wybrał "ship when ready" bez twardego terminu. Zaktualizować frontmatter jeśli pojawi się zewnętrzna presja daty. Block: nie.

---
project: go-hotel
context_type: greenfield
updated: 2026-06-17
checkpoint:
  current_phase: 8
  phases_completed: [1, 2, 3, 4, 5, 6, 7]
  frs_drafted: 8
  quality_check_status: accepted
---

## Vision & Problem Statement

Istniejące gry hotelowe na mobile są przeładowane reklamami i nie dają czystej przyjemności z rozgrywki. Gracz casual nie ma dobrej alternatywy na krótkie sesje (2–5 min) bez przerywania co chwilę reklamami.

Go-hotel to prosta gra mobilna/webowa bez reklam, w której jesteś właścicielem hotelu — budujesz go od zera, zatrudniasz pracowników, automatyzujesz operacje i rozwijasz go w czasie. Projekt osobisty z ambicją publicznego releasu.

**Typ bólu:** brakująca jakość (istniejące gry w gatunku są złe z powodu nadmiaru reklam)

**Insight:** gracz, który sam buduje grę z AI, wie od środka co psuje doświadczenie — i może tego uniknąć od pierwszego dnia.

**Gatunek:** hybryd idle + tycoon — hotel działa pasywnie między sesjami, ale w dłuższych sesjach gracz aktywnie zarządza i podejmuje decyzje.

## User & Persona

**Persona główna:** Casual gracz mobilny — ty sam oraz osoby o podobnym profilu.

- Wiek: 20–40 lat
- Kontekst użycia: krótkie sesje 2–5 min (toaleta, kolejka, przerwa w pracy)
- Oczekiwania: szybki progress widoczny w jednej sesji; brak reklam; gra "czeka" na gracza gdy wróci
- Motywacja: satysfakcja z budowania i rozwijania czegoś własnego
- Frustracja z istniejących gier: agresywna monetyzacja, przerywające reklamy

## Access Control

- Brak wymaganego logowania — gra startuje natychmiast, zapis lokalny domyślnie
- Opcjonalne konto (Google/Apple Sign-In) odblokowuje synchronizację zapisu między urządzeniami
- Model ról: płaski — jeden typ gracza, każdy ma swój własny hotel
- Najdrobniejsza wersja MVP: zapis lokalny tylko, bez chmury; synchronizacja to feature v2

## Success Criteria

### Primary
MVP flow (dowód że gra działa):
1. Gracz otwiera aplikację — od razu widok hotelu 2D, bez menu
2. Recepcja sygnalizuje czekającego gościa (pulsująca ikona)
3. Gracz klika "przyjmij gościa" → postać idzie do recepcji (animacja)
4. Gość przypisany do pokoju, idzie sam
5. Po kilku sekundach pokój jest "brudny"
6. Gracz klika "sprzątaj pokój X" → postać idzie, animuje sprzątanie
7. Pokój gotowy → kolejny gość może wejść
8. Gracz widzi narastający przychód i może kupić odblokowanie drugiego pokoju

### Secondary
- Drugi pokój do odblokowania za zarobione pieniądze (pierwsza pętla progresji: zarabiasz → kupujesz → więcej zarabiasz)

### Guardrails
- Zapis postępów musi przeżyć odświeżenie strony i zamknięcie aplikacji (localStorage / zapis serwera)
- Zero reklam w jakiejkolwiek formie

## Timeline
- mvp_weeks: null (brak twardego deadline — ship gdy gotowe)
- after_hours_only: true
- Scope down: wersja 2D z klik-na-zadanie, nie izometryczne 2.5D z aktywnym sterowaniem

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

## Product Framing

- product_type: web-app (gra przeglądarkowa)
- target_scale: small (ja + kilka osób na MVP)
- timeline_budget:
  - mvp_weeks: null (ship gdy gotowe)
  - hard_deadline: null
  - after_hours_only: true

## Non-Goals

- Brak trybu multiplayer ani rankingów — każdy gracz ma swój prywatny hotel, zero interakcji między graczami
- Brak jakiejkolwiek monetyzacji (IAP, subscription, reklamy) — MVP bezpłatne; decyzja po walidacji gry
- Brak eventów sezonowych / specjalnych — bez świątecznych content drops ani time-limited challenges
- Brak systemu ocen gości — gość płaci i wychodzi, zero gwiazdek ani recenzji w MVP

## Forward: tech-stack

- Musi obsługiwać mobile web jako primary target
- Ambicja: deploy do Play Store i App Store z minimalną zmianą codebase (Capacitor/PWA wrapper) — v1.1
- Gra 2D (nie izometryczne 2.5D) — potrzebna biblioteka renderingu 2D (Phaser.js lub równoważna)
- Offline-first: LocalStorage zapis stanu gry; opcjonalny cloud sync przez opcjonalne konto

## Business Logic

Hotel generuje przychód za każdego obsłużonego gościa. Liczba wolnych, czystych pokoi to jedyna zmienna która skaluje przychód — gracz maksymalizuje ją przez szybkie sprzątanie lub delegowanie do pracownika. Odblokowanie kolejnych pokoi wymaga osiągnięcia progu gotówki, tworząc pętlę: obsłuż gości → zarabiaj → odblokuj → obsłuż więcej gości.

Gra nie ocenia jakości obsługi (brak systemu ocen gości w MVP) — liczy się wyłącznie przepustowość: ile pokoi działa, jak szybko wracają do stanu "wolny".

## Non-Functional Requirements

- Czas uruchomienia gry < 3 sekundy na średnio szybkim urządzeniu mobilnym (mierzone jako czas do pierwszej interaktywnej klatki)
- Podstawowa rozgrywka działa offline; stan gry synchronizowany gdy internet jest dostępny
- MVP: mobile web (przeglądarka na telefonie); codebase musi pozwalać na deploy do Play Store i App Store z minimalną zmianą (Capacitor/PWA wrapper) — nice-to-have na MVP, wymaganie dla v1.1
- Zero reklam — żadna forma reklamy w interfejsie gry

## User Stories

### US-01: Cykl obsługi gościa
- **Given:** hotel jest otwarty, jest co najmniej jeden wolny, czysty pokój
- **When:** gracz klika na pulsującą recepcję i akceptuje gościa
- **Then:** gość idzie do pokoju, po odejściu pokój jest oznaczony jako brudny, gracz może go posprzątać, po sprzątaniu pokój wraca do stanu "wolny" i przychód gracza rośnie

# Dokumentacja produktu: Platforma do etycznych kampanii phishingowych + integracja e-learning

## 1. Cel dokumentu
Ten dokument opisuje plan i pełną specyfikację serwisu SaaS do **wysyłki oraz zarządzania etycznymi kampaniami phishingowymi** dla firm.
Platforma ma wspierać:
- zakładanie i obsługę **kont firmowych (multi-tenant)**,
- planowanie, uruchamianie i raportowanie kampanii,
- automatyczne przypisanie użytkownika do szkolenia cyberbezpieczeństwa w Twojej istniejącej aplikacji e-learningowej po „złapaniu się” na phishing.

---

## 2. Wizja produktu

### 2.1 Problem biznesowy
Firmy potrzebują regularnych, mierzalnych testów odporności pracowników na socjotechnikę. Sam test bez szkolenia nie daje trwałych efektów.

### 2.2 Rozwiązanie
System realizuje cykl:
1. Firma tworzy konto i strukturę organizacji.
2. Administrator planuje kampanię phishingową.
3. Pracownicy otrzymują symulowane wiadomości.
4. System rejestruje zachowania (otwarcie, kliknięcie, wpisanie danych testowych, zgłoszenie incydentu).
5. Osoby podatne są automatycznie przypisywane do szkoleń w aplikacji e-learning.
6. Menedżer otrzymuje raporty i KPI.

### 2.3 Definicja sukcesu (KPI)
- Spadek współczynnika kliknięć (Click Rate) miesiąc do miesiąca.
- Wzrost współczynnika zgłaszania podejrzanych wiadomości (Report Rate).
- Wzrost ukończeń przypisanych szkoleń.
- Skrócenie czasu reakcji od kliknięcia do rozpoczęcia szkolenia.

---

## 3. Zakres systemu

### 3.1 Moduły główne
1. **Identity & Access** – rejestracja kont firmowych, logowanie, RBAC, SSO (opcjonalnie).
2. **Tenant Management** – separacja danych między firmami, branding, limity planów.
3. **Campaign Builder** – kreator kampanii, szablony wiadomości i landing page’y.
4. **Audience Management** – import użytkowników (CSV/API), grupy i segmenty.
5. **Simulation Engine** – harmonogram wysyłek, throttling, śledzenie zdarzeń.
6. **Tracking & Detection** – eventy: delivery/open/click/submit/report.
7. **Training Assignment** – integracja z e-learningiem, automatyczne przypisania.
8. **Reporting & Analytics** – dashboardy, wyniki kampanii, porównania działów.
9. **Compliance & Audit** – logi audytowe, polityki retencji, anonimizacja.
10. **Billing (opcjonalnie)** – plany, licencje, rozliczenia.

### 3.2 Zakres MVP
- Konto firmowe + role.
- Import pracowników CSV.
- Tworzenie kampanii z gotowego szablonu.
- Wysyłka kampanii i podstawowe śledzenie kliknięć.
- Integracja z e-learningiem: przypisanie kursu po kliknięciu/link-submit.
- Raport kampanii + eksport CSV.

### 3.3 Poza MVP
- Zaawansowane scenariusze adaptive phishing.
- AI do generowania treści kampanii.
- A/B testy wielowariantowe.
- Integracje SIEM/SOAR.

---

## 4. Role i uprawnienia (RBAC)

### 4.1 Role systemowe
- **Super Admin (platforma)** – zarządzanie tenantami, planami, support.
- **Company Owner** – pełna administracja kontem firmowym.
- **Security Admin** – tworzenie kampanii, zarządzanie odbiorcami, raporty.
- **Manager** – wgląd w wyniki zespołu, bez konfiguracji krytycznej.
- **Employee** – odbiorca symulacji i kursów.

### 4.2 Macierz uprawnień (skrót)
- Tworzenie kampanii: Owner/Security Admin.
- Publikacja kampanii: Owner/Security Admin.
- Zarządzanie użytkownikami: Owner/Security Admin.
- Podgląd raportów globalnych: Owner/Security Admin.
- Podgląd raportów zespołu: Manager.
- Odbiór i realizacja kursów: Employee.

---

## 5. Procesy biznesowe (E2E)

### 5.1 Onboarding firmy
1. Rejestracja firmy (nazwa, domena, liczba pracowników, branża).
2. Weryfikacja domeny e-mail (DNS TXT lub link).
3. Utworzenie pierwszego administratora.
4. Konfiguracja podstawowa (strefa czasowa, polityka retencji, branding).
5. Import pracowników i struktur (działy, managerowie).

### 5.2 Tworzenie kampanii phishingowej
1. Wybór celu kampanii (np. credential harvesting, attachment test, QR).
2. Wybór grupy odbiorców.
3. Wybór szablonu i personalizacji (imię, dział, kontekst firmowy).
4. Konfiguracja harmonogramu i okna wysyłki.
5. Wybór „strony edukacyjnej po kliknięciu” lub silent mode.
6. Definicja automatycznej akcji: przypisz szkolenie X po zdarzeniu Y.
7. Preview i test wewnętrzny.
8. Start kampanii.

### 5.3 Obsługa „złapania się” na phishing
1. Użytkownik klika link (lub wpisuje dane testowe na landing page).
2. System zapisuje zdarzenie i klasyfikuje poziom ryzyka.
3. Reguła automatyzacji uruchamia przypisanie kursu w aplikacji e-learning.
4. Pracownik dostaje powiadomienie (mail/in-app) o obowiązkowym szkoleniu.
5. Dashboard aktualizuje status (przypisano / rozpoczęto / ukończono).

### 5.4 Raportowanie i zamknięcie kampanii
1. Zbieranie danych do końca okna kampanii.
2. Obliczenie KPI per dział / lokalizacja / seniority.
3. Wygenerowanie rekomendacji działań naprawczych.
4. Eksport danych i archiwizacja.

---

## 6. Integracja z Twoją aplikacją e-learning

### 6.1 Cel integracji
Po zdarzeniu phishingowym system ma automatycznie przypisać odpowiedni kurs cyberbezpieczeństwa i monitorować realizację.

### 6.2 Model integracji (rekomendowany)
**Webhook + REST API (dwukierunkowo):**
- Platforma phishingowa wysyła event `user_compromised` do e-learningu.
- E-learning odpowiada `assignment_created` z `assignment_id`.
- E-learning wysyła statusy: `started`, `completed`, `failed`, `overdue`.

### 6.3 Minimalny kontrakt API

#### Endpoint 1: Przypisanie szkolenia
`POST /api/v1/training-assignments`

Przykładowy payload:
```json
{
  "tenant_id": "t_123",
  "employee_external_id": "emp_8841",
  "trigger": "phishing_click",
  "campaign_id": "camp_2026_04_01",
  "course_code": "CYBER-PHISH-101",
  "deadline": "2026-05-01T23:59:59Z"
}
```

#### Endpoint 2: Aktualizacja statusu szkolenia (callback)
`POST /api/v1/training-assignments/status`

Przykładowy payload:
```json
{
  "assignment_id": "as_7781",
  "employee_external_id": "emp_8841",
  "status": "completed",
  "completed_at": "2026-04-22T11:05:00Z",
  "score": 92
}
```

### 6.4 Identyfikacja użytkownika
- Rekomendowane: stałe `employee_external_id` + `tenant_id`.
- Fallback: e-mail służbowy (mniej bezpieczne przy zmianach adresów).

### 6.5 Retry i niezawodność
- Idempotency-Key na żądaniach przypisania.
- Retry z backoff (np. 1m, 5m, 15m, 1h).
- Dead-letter queue dla błędnych payloadów.

---

## 7. Architektura logiczna

### 7.1 Proponowane komponenty
- **Frontend Admin Portal** (React/Vue/Angular).
- **API Gateway** (auth, rate-limit, routing).
- **Auth Service** (JWT/OIDC/SAML opcjonalnie).
- **Campaign Service**.
- **Delivery Service** (SMTP provider abstraction).
- **Tracking Service** (pixel/link tracking).
- **Rules Engine** (automatyzacje przypisań).
- **Integration Service** (e-learning webhooks/API).
- **Reporting Service** (agregacje, dashboard).
- **Audit Service**.
- **Data Stores**: PostgreSQL + Redis + Object Storage + Queue (np. RabbitMQ/SQS/Kafka).

### 7.2 Multi-tenant
- Wspólna baza z `tenant_id` + RLS lub osobne schematy per tenant.
- Silna izolacja logiczna i audyt dostępu do danych.

### 7.3 Bezpieczeństwo
- TLS 1.2+ wszędzie.
- Szyfrowanie at-rest (KMS).
- MFA dla adminów.
- RBAC + zasada najmniejszych uprawnień.
- Podpisywanie webhooków (HMAC).

---

## 8. Model danych (skrót)

### 8.1 Encje
- `tenants`
- `users`
- `roles`
- `departments`
- `campaigns`
- `campaign_templates`
- `campaign_recipients`
- `phishing_events`
- `training_assignments`
- `training_progress`
- `audit_logs`

### 8.2 Przykład relacji
- `tenant` 1..N `users`
- `campaign` 1..N `campaign_recipients`
- `campaign_recipient` 1..N `phishing_events`
- `user` 1..N `training_assignments`

---

## 9. Raportowanie i metryki

### 9.1 Metryki operacyjne
- Delivery Rate
- Open Rate
- Click Rate
- Submission Rate
- Report Rate
- Time-to-Report

### 9.2 Metryki szkoleniowe
- Assignment Rate po kampanii
- Start Rate
- Completion Rate
- Average Completion Time
- Re-offender Rate (powtórne złapanie)

### 9.3 Dashboardy
- Widok zarządczy (cała firma).
- Widok działów / lokalizacji.
- Ranking ryzyka zespołów.
- Trend kwartalny i porównanie kampanii.

---

## 10. Wymagania niefunkcjonalne

### 10.1 Wydajność
- Obsługa kampanii do 100k odbiorców na tenant (skokowo).
- Przetwarzanie eventów near-real-time (< 10 s dla dashboardu bieżącego).

### 10.2 Dostępność
- SLA: 99.9% (po MVP).
- Backups dzienne + point-in-time recovery.

### 10.3 Skalowalność
- Horyzontalne skalowanie Tracking i Delivery Service.
- Kolejkowanie zadań do wysyłek i webhooków.

### 10.4 Obserwowalność
- Logi strukturalne.
- Metryki (Prometheus/OpenTelemetry).
- Alerting (SLO breach, spike błędów webhooków).

---

## 11. Compliance, etyka i aspekty prawne

> Uwaga: to nie jest porada prawna – warto skonsultować wdrożenie z prawnikiem/inspektorem ochrony danych.

- Jasny cel bezpieczeństwa i polityka wewnętrzna testów.
- Informowanie pracowników o programie podnoszenia świadomości (bez ujawniania scenariuszy).
- Minimalizacja danych osobowych i polityka retencji.
- Pseudonimizacja w raportach zarządczych tam, gdzie możliwe.
- Kontrola dostępu do danych wrażliwych i pełen audit trail.

---

## 12. Roadmapa wdrożenia

### Faza 0 – Discovery (1–2 tyg.)
- Warsztat wymagań.
- Ustalenie integracji z e-learningiem (API contract).
- Definicja KPI i polityk prawnych.

### Faza 1 – MVP (6–10 tyg.)
- Konta firmowe + RBAC.
- Import użytkowników i grup.
- Kampanie z szablonów.
- Tracking click/submit.
- Integracja z e-learning i status ukończenia.
- Dashboard podstawowy + eksport CSV.

### Faza 2 – Harden & Scale (4–8 tyg.)
- SSO/SAML, MFA, audit rozszerzony.
- Bardziej granularne raporty.
- Zaawansowane reguły automatyzacji.

### Faza 3 – Enterprise Features
- Branding white-label.
- Rozbudowany billing i marketplace integracji.
- Benchmarki międzyokresowe i predykcja ryzyka.

---

## 13. Kryteria akceptacji MVP
1. Firma może utworzyć konto i zaprosić min. 2 role administracyjne.
2. Administrator może zaimportować listę pracowników z CSV.
3. Administrator uruchomi kampanię dla wybranej grupy.
4. System poprawnie zapisze zdarzenia kliknięcia i submit.
5. Po zdarzeniu `phishing_click` kurs zostanie przypisany w e-learningu.
6. Status ukończenia kursu wróci do platformy i będzie widoczny w raporcie.
7. Możliwy eksport raportu kampanii do CSV.

---

## 14. Proponowana struktura repo dokumentacji

```text
/docs
  /product
    vision.md
    requirements.md
    roadmap.md
  /architecture
    system-design.md
    data-model.md
    integrations-elearning.md
  /operations
    security.md
    compliance.md
    runbook.md
```

---

## 15. Następne kroki (praktycznie)
1. Potwierdź format identyfikatorów użytkownika pomiędzy systemami.
2. Spisz kontrakt API e-learning (OpenAPI/Swagger).
3. Ustal katalog kursów i mapowanie „typ zdarzenia → kurs”.
4. Ustal politykę SLA dla webhooków i retry.
5. Zrób pilotaż na 1–2 działach przez 4 tygodnie.
6. Na bazie wyników skaluj na całą organizację.

---

Jeśli chcesz, w kolejnym kroku mogę od razu przygotować:
- gotowe **OpenAPI 3.0** dla integracji phishing ↔ e-learning,
- szablony tabel SQL pod MVP,
- checklistę wdrożenia pod produkcję.

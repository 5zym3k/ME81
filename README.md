# ME81H: termostat Tuya na ESPHome

Konfiguracja ESPHome dla termostatów ME81H (Tuya MCU, produkt `gogb05wrtredz3bs`) z Home Assistantem.

## Struktura

| Plik | Opis |
|---|---|
| `common/me81h.yaml` | wspólna konfiguracja (package) |
| `common/secrets.yaml` | przekierowanie do `../secrets.yaml`; `!secret` w pakiecie szuka sekretów w katalogu pakietu |
| `secrets.yaml.example` | wzór sekretów, skopiuj do `secrets.yaml` |

Każdy termostat to krótki plik obok katalogu `common/`, np. `termostat-salon.yaml`:

```yaml
substitutions:
  name: thermostat01
  friendly_name: Termostat-Salon
  board: esp01_1m              # TYWE3S: esp01_1m, ESP-12F: esp12e
  status_pin_inverted: "true"  # TYWE3S: "true", ESP-12F: "false"

packages:
  me81h: !include common/me81h.yaml
```

## Mapa datapointów

| DP | Typ | Znaczenie | Uwagi |
|---|---|---|---|
| 1 | bool | włączony / wyłączony | |
| 2 | enum | tryb: 0 Auto, 1 Ręczny | |
| 10 | bool | antyzamarzanie | |
| 16 | int | temperatura zadana | ×1, pełne stopnie |
| 19 | int | temperatura maksymalna (limit) | zapis działa (sprawdzone); możliwe, że to limit podłogi, nie nastawy |
| 24 | int | temperatura bieżąca | ×0,1; czujnik ma krok 0,5°C |
| 27 | int | kalibracja | MCU raportuje sam, **bez** `datapoint_hidden` |
| 36 | enum | stan grzania: 0 grzeje, 1 idle | sprawdzone na dwóch sztukach |
| 38 | raw | harmonogram tygodniowy | nieobsługiwany |
| 40 | bool | blokada panelu | |
| 43 | enum | czujnik: 0 wewn., 1 zewn., 2 oba | |
| 45 | bitmask | kod błędu, 0 = OK | |
| 101 | int | próg aktywacji (histereza) | **bez** `datapoint_hidden` |
| 102 | enum | harmonogram dni: 0 wył., 1 Pn-Pt, 2 Pn-Sob, 3 Pn-Ndz | |

**DP 26 nie istnieje.** Encja „Temperatura minimalna” na DP 26 nic nie wysyła do MCU.

## Status pin, czyli ikonka WiFi i czas na panelu

MCU zgłasza `GPIO Configuration: status: pin 14, reset: pin 0`, więc ESPHome sygnalizuje sieć stanem GPIO14 zamiast komendy `WIFI_STATE` przez UART.

- **Oryginalny TYWE3S:** wymaga `inverted: true`. Bez tego panel nie pokazuje WiFi i nie ustawia godziny, bo MCU uważa, że jest offline. Potwierdzone na sprzęcie (ESPHome 2026.10.2).
- **Moduł wymieniony na ESP-12F:** działał z `inverted: false` (ESPHome 2025.12.5). Jedna taka sztuka raportowała dziwne `pin 2, reset 17`.

**Uwaga przy aktualizacji do ESPHome ≥ 2026.10:** przy niezgodnym pinie ESPHome i tak steruje pinem z YAML-a, a starsze wersje przechodziły wtedy w „limited mode”. Po aktualizacji sztuki z ESP-12F sprawdź, czy ikonka WiFi nadal działa.

## Inne uwagi

- `logger: baud_rate: 0`, bo UART0 obsługuje Tuya MCU.
- Selecty bez `optimistic`: HA pokazuje faktyczny stan MCU.
- `visual.min_temperature: 5`, żeby z HA dało się zejść do temperatury antyzamarzania.

## Wgrywanie

```bash
cp secrets.yaml.example secrets.yaml   # uzupełnij
esphome run termostat-salon.yaml
```

W Home Assistant (dodatek ESPHome) skopiuj `common/` oraz pliki urządzeń do `/config/esphome/`. Sekrety są w tamtejszym `secrets.yaml`, dopisz `ap_password`.

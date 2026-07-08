# arduino

Firmware du capteur physique température/humidité (Brésil) du projet FutureKawa.

## Fonctionnalités

- **Lecture DHT22** : relevé de la température et de l'humidité toutes les 10 secondes
  (`sketch_may6a/sketch_may6a.ino`, pin `D4`, capteur `DHT22`).
- **Sortie JSON sur le port série** : chaque relevé est publié en JSON
  (`{"temperature": ..., "humidity": ...}`) sur le port série à 9600 bauds, prêt à être
  consommé par un pont série (voir `MQTT_Broker/mqtt.py` et `MQTT_Broker/session_lot.py`
  dans le dépôt principal, qui republient ces relevés vers MQTT/la base de données).
- **Gestion des erreurs capteur** : en cas d'échec de lecture du DHT22, envoie
  `{"error": "Failed to read from DHT sensor"}` sur le port série et retente après 1 minute.
- **Intégration continue** : le workflow GitHub Actions (`.github/workflows/ci.yml`)
  installe le core `arduino:avr` et la librairie `DHT sensor library`, puis compile le
  sketch (`arduino-cli compile --fqbn arduino:avr:uno`) à chaque push/pull request.

## Matériel

- Carte Arduino Uno R3
- Capteur DHT22 câblé sur la pin numérique 4

## Compilation & upload

Depuis la racine du monorepo (voir `COMMANDS.md`), avec `arduino-cli` :

```bash
./arduino-cli core install arduino:avr
./arduino-cli lib install "DHT sensor library"
./arduino-cli compile --fqbn arduino:avr:uno arduino/sketch_may6a/
./arduino-cli upload -p /dev/ttyACM0 --fqbn arduino:avr:uno arduino/sketch_may6a/
```

⚠️ Un seul processus peut lire le port série (`/dev/ttyACM0`) à la fois : ne pas lancer
`arduino-cli monitor` en même temps que `MQTT_Broker/mqtt.py` ou
`MQTT_Broker/session_lot.py`.

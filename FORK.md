# Fork-Hinweise

Dieser Fork existiert wegen eines Fixes, der upstream noch nicht gemerged ist:
**[wez/govee2mqtt#716](https://github.com/wez/govee2mqtt/pull/716)** — modellspezifische
Szenen-Präfixe, ohne die Szenen auf einer H6022 stillschweigend wirkungslos bleiben.

## Branches

| Branch | Zweck |
|---|---|
| `main` | exakter Spiegel von `wez/govee2mqtt` — **nicht anfassen**, damit der PR sauber rebasebar bleibt |
| `fix/h6022-scene-prefix` | der PR-Branch, genau ein Commit über `main` |
| `ha-addon` | Default-Branch: der Fix **plus** die Umstellung auf eigene Images. Von hier installiert Home Assistant. |

## Installation in Home Assistant

1. *Einstellungen → Add-ons → Add-on Store → ⋮ → Repositories* → `https://github.com/knows-cloud/govee2mqtt`
2. „Govee to MQTT Bridge (Fork)" installieren (Slug `govee2mqtt-fork`, bewusst anders als upstream,
   damit beide parallel installiert sein können)
3. Optionen übertragen: `govee_email`, `govee_password`, `govee_api_key`, `temperature_scale`
4. **Erst das alte Add-on stoppen, dann das neue starten.** Zwei Bridges auf demselben
   MQTT-Broker streiten sich um dieselben Discovery-Topics.

## Neue Version bauen

Die Version in `addon/config.yaml` ist die einzige Quelle der Wahrheit: Der Builder
veröffentlicht genau dieses Tag, und der Supervisor fragt genau danach. `scripts/apply-tag.sh`
läuft absichtlich **nicht** mehr in der CI — es hätte die Datei nur im Workspace geändert und
so ein Tag erzeugt, das im Repo nirgends steht.

1. `version:` in `addon/config.yaml` erhöhen
2. nach `ha-addon` pushen → CI baut `ghcr.io/knows-cloud/govee2mqtt-{arch}:<version>`
3. in HA das Add-on aktualisieren

## Wenn upstream den PR merged

Dann wird dieser Fork überflüssig:

```
git fetch upstream && git checkout main && git merge --ff-only upstream/main && git push origin main
```

Danach in HA auf das offizielle Add-on zurückwechseln (Optionen übertragen, altes stoppen),
dieses Repository aus den Add-on-Repositories entfernen und den Default-Branch
wieder auf `main` stellen.

## Abweichungen von upstream auf `ha-addon`

- `IMAGE`/`image:`/Add-on-Dockerfile zeigen auf `ghcr.io/knows-cloud/...` statt `ghcr.io/wez/...`
- Build läuft auch auf Push nach `ha-addon` und per `workflow_dispatch`, kein Release-Tag nötig
- `cosign` entfernt: Der Builder validiert damit die Home-Assistant-Basis-Images, und die sind
  derzeit unsigniert („no signatures found" → „Invalid base image"). Betrifft upstream genauso.
  Für ein eigenes Add-on-Repository ist die Signatur optional.

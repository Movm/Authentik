# Authentik Deployment für Coolify

## Übersicht
Dieses Repository enthält die Docker Compose Konfiguration für Authentik, optimiert für den Einsatz mit Coolify.

## Services
- **authentik-server**: Hauptserver für Authentik
- **authentik-worker**: Worker für Background-Tasks  
- **postgresql**: PostgreSQL Datenbank
- **redis**: Redis Cache

## Ordnerstruktur
```
authentik-deploy/
├─ docker-compose.yml      # Docker Compose Konfiguration
├─ README.md              # Diese Dokumentation
├─ branding/              # Custom Branding Assets
│   ├─ logo.svg
│   ├─ favicon.png
│   └─ custom.css
└─ templates/             # Custom Templates
    ├─ email/
    │   └─ custom-verification.html
    └─ flows/
        └─ custom-login-flow.yaml
```

## Deployment mit Coolify

1. Repository in Coolify importieren
2. Environment-Variablen in Coolify UI konfigurieren:
   - `SERVICE_USER_POSTGRESQL`
   - `SERVICE_PASSWORD_POSTGRESQL`
   - `SERVICE_PASSWORD_64_AUTHENTIKSERVER`
   - Email-Konfiguration (optional)

3. Service deployen

## Volumes
- `authentik-db`: PostgreSQL Daten
- `redis`: Redis Daten
- `./media`: Authentik Media Files
- `./custom-templates`: Custom Templates
- `./certs`: Zertifikate

## Erste Schritte
Nach dem Deployment ist Authentik über Port 9000 erreichbar. Der Standard-Admin Account wird automatisch erstellt.

## Customization
- Branding-Assets in `branding/` ablegen
- Custom Templates in `templates/` erstellen
- CSS-Anpassungen in `branding/custom.css` 
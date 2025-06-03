# 🚀 Grünerator Authentik Setup Guide

Diese Anleitung führt dich durch die vollständige Konfiguration von Authentik für die Grünerator-Anwendung mit allen benötigten Flows, Policies und Mappings.

## 📋 Voraussetzungen

- ✅ Authentik läuft und ist erreichbar
- ✅ Admin-Zugang zu Authentik (akadmin)
- ✅ Grundkonfiguration aus `authentik-config.txt` ist angewendet

## 🔧 Installation Schritt-für-Schritt

### 1. Import der Konfigurationsdateien

Die Konfiguration ist in mehrere YAML-Dateien aufgeteilt für bessere Übersichtlichkeit:

```bash
# Reihenfolge ist wichtig!
1. gruenerator-property-mappings.yaml  # Property Mappings zuerst
2. gruenerator-flows.yaml              # Flows und Stages
3. gruenerator-policies.yaml           # Policies und Bindings
4. gruenerator-complete-config.yaml    # Provider, Application, Groups
```

### 2. Import in Authentik

1. **Login** in Authentik Admin Interface: `https://deine-domain:9443/if/admin/`
2. **Navigiere** zu `System` > `Flows & Stages` > `Flows`
3. **Klick** auf `Import` (oben rechts)
4. **Importiere nacheinander** in dieser Reihenfolge:

   **Schritt 1: Property Mappings**
   ```
   Datei: gruenerator-property-mappings.yaml
   ✓ Upload und Import
   ```

   **Schritt 2: Flows und Stages**
   ```
   Datei: gruenerator-flows.yaml
   ✓ Upload und Import
   ```

   **Schritt 3: Policies**
   ```
   Datei: gruenerator-policies.yaml
   ✓ Upload und Import
   ```

   **Schritt 4: Complete Configuration**
   ```
   Datei: gruenerator-complete-config.yaml
   ✓ Upload und Import
   ```

## 🎯 Was wird installiert?

### 🌊 **Flows**
- `gruenerator-login-flow` - Authentication Flow
- `gruenerator-registration-flow` - Enrollment Flow  
- `gruenerator-unenrollment-flow` - Account Deletion Flow
- `gruenerator-invalidation-flow` - Logout Flow

### 🎭 **Stages**
- **Identification Stage** - Login mit Username/Email
- **Password Stage** - Passwort-Authentifizierung
- **User Login Stage** - Session-Management
- **User Write Stage** - User-Erstellung mit Gruppenzuordnung
- **Prompt Stage** - Registrierungsformular
- **Email Stage** - E-Mail-Verifikation
- **User Delete Stage** - Account-Löschung
- **User Logout Stage** - Session-Beendigung

### 🔒 **Policies**
- **Password Policy** - Sichere Passwort-Richtlinien
- **Expression Policies** - Validierung und Gruppenzuordnung
- **Auto Group Assignment** - Automatische Gruppen-Zuordnung

### 📋 **Property Mappings**
- **Profile Scope** - Benutzerprofile mit deutschen Lokalisierung
- **Email Scope** - E-Mail und Verifikationsstatus
- **OpenID Scope** - Standard OIDC Claims
- **Offline Access** - Refresh Token Support
- **Groups Mapping** - Erweiterte Gruppen-Informationen
- **User Attributes** - Zusätzliche User-Metadaten
- **Permissions Mapping** - Rollen-basierte Berechtigungen

### 👥 **Groups**
- **Grünerator Users** - Standard-Benutzer (read, write, profile_edit)
- **Grünerator Editors** - Content-Editoren (+ content_edit, moderate)  
- **Grünerator Admins** - Administratoren (+ admin, manage_users, manage_settings)

### 🔐 **Provider & Application**
- **OAuth2/OpenID Provider**: `gruenerator-provider`
  - Client ID: `gruenerator_oidc_client`
  - Alle notwendigen Scopes und Mappings
- **Application**: `gruenerator`
  - Verbunden mit Provider
  - Konfiguriert für OIDC-Authentication

## ⚙️ Nach dem Import

### 1. Provider-Credentials notieren

Nach dem Import findest du die Provider-Informationen unter:
`Applications` > `Providers` > `gruenerator-provider`

**Notiere dir:**
- ✅ **Client ID**: `gruenerator_oidc_client`
- ✅ **Client Secret**: [wird automatisch generiert]
- ✅ **Issuer URL**: `https://deine-domain/application/o/gruenerator-provider/`

### 2. Redirect URIs anpassen

**Wichtig**: Passe die Redirect URIs an deine tatsächliche Domain an:

1. Gehe zu `Applications` > `Providers` > `gruenerator-provider`
2. Bearbeite die **Redirect URIs**:
   ```
   https://deine-echte-domain.com/api/auth/callback
   http://localhost:3000/api/auth/callback  # Für Development
   ```

### 3. E-Mail-Konfiguration (optional)

Passe die E-Mail-Settings in `gruenerator-email-stage` an:
- **SMTP Server**: Dein E-Mail-Provider
- **From Address**: `noreply@deine-domain.com`
- **Templates**: Können in `authentik-deploy/templates/email/` angepasst werden

### 4. Branding anpassen (optional)

Das Default-Branding wird auf "Grünerator SSO" gesetzt. Anpassen unter:
`System` > `Brands` > `default`

## 🔗 Integration in deine App

### Environment Variables für deine Grünerator-App:

```env
# OAuth2/OIDC Configuration
NEXTAUTH_URL=https://deine-domain.com
NEXTAUTH_SECRET=dein-nextauth-secret

# Authentik Provider
AUTH_AUTHENTIK_ID=gruenerator_oidc_client
AUTH_AUTHENTIK_SECRET=der-generierte-client-secret
AUTH_AUTHENTIK_ISSUER=https://deine-authentik-domain/application/o/gruenerator-provider/

# Optional: Custom Claims
AUTH_AUTHENTIK_SCOPE=openid profile email offline_access
```

### NextAuth.js Configuration Beispiel:

```typescript
// pages/api/auth/[...nextauth].ts
import { AuthOptions } from "next-auth"
import { JWT } from "next-auth/jwt"

export const authOptions: AuthOptions = {
  providers: [
    {
      id: "authentik",
      name: "Grünerator Login",
      type: "oauth",
      clientId: process.env.AUTH_AUTHENTIK_ID!,
      clientSecret: process.env.AUTH_AUTHENTIK_SECRET!,
      issuer: process.env.AUTH_AUTHENTIK_ISSUER!,
      wellKnown: `${process.env.AUTH_AUTHENTIK_ISSUER!}/.well-known/openid-configuration`,
      authorization: {
        params: {
          scope: "openid profile email offline_access"
        }
      },
      client: {
        id_token_signed_response_alg: "RS256"
      },
      profile(profile) {
        return {
          id: profile.sub,
          name: profile.name,
          email: profile.email,
          image: profile.picture,
          groups: profile.groups || [],
          permissions: profile.permissions || [],
          roles: profile.roles || []
        }
      }
    }
  ],
  callbacks: {
    async jwt({ token, account, profile }) {
      if (account && profile) {
        token.groups = profile.groups
        token.permissions = profile.permissions
        token.roles = profile.roles
        token.accessToken = account.access_token
        token.refreshToken = account.refresh_token
      }
      return token
    },
    async session({ session, token }) {
      session.user.groups = token.groups as string[]
      session.user.permissions = token.permissions as string[]
      session.user.roles = token.roles as string[]
      return session
    }
  }
}
```

## 🧪 Testing

### 1. Flow-Tests

**Login Flow:**
```
1. Gehe zu: https://deine-authentik-domain/if/flow/gruenerator-login-flow/
2. Teste Login mit Username/Email + Password
3. Überprüfe Session-Erstellung
```

**Registration Flow:**
```
1. Gehe zu: https://deine-authentik-domain/if/flow/gruenerator-registration-flow/
2. Fülle Registrierungsformular aus
3. Überprüfe E-Mail-Verifikation
4. Prüfe automatische Gruppenzuordnung
```

### 2. OIDC-Endpoints testen

```bash
# Discovery Endpoint
curl https://deine-authentik-domain/application/o/gruenerator-provider/.well-known/openid-configuration

# Authorization Endpoint (im Browser)
https://deine-authentik-domain/application/o/authorize/?client_id=gruenerator_oidc_client&response_type=code&scope=openid+profile+email&redirect_uri=https://deine-domain/api/auth/callback
```

## 🔍 Troubleshooting

### Häufige Probleme:

**1. Import-Fehler "KeyOf not found"**
```
❌ Problem: Dependencies nicht in richtiger Reihenfolge importiert
✅ Lösung: Reihenfolge beachten (Property Mappings → Flows → Policies → Complete)
```

**2. "Source nicht gefunden"**
```
❌ Problem: gruenerator-login-source existiert nicht
✅ Lösung: Complete Config importieren, dann Source manuell erstellen
```

**3. "Client Secret fehlt"**
```
❌ Problem: Provider wurde nicht korrekt erstellt
✅ Lösung: Provider bearbeiten, neues Client Secret generieren
```

**4. "Redirect URI Mismatch"**
```
❌ Problem: URLs stimmen nicht überein
✅ Lösung: Redirect URIs in Provider anpassen
```

### Debug-Informationen:

**Logs anzeigen:**
```bash
# Authentik Container Logs
docker logs authentik-server

# Policy Evaluierung debuggen
# System > Events > Logs filtern nach Policy
```

**Flow-Debug:**
- `System` > `Events` > `Flow Execution Logs`
- `Applications` > `Providers` > `gruenerator-provider` > `Preview`

## 📚 Weiterführende Dokumentation

- [Authentik OAuth2/OIDC Provider](https://docs.goauthentik.io/docs/providers/oauth2/)
- [Authentik Flow & Stage System](https://docs.goauthentik.io/docs/flows/)
- [Authentik Property Mappings](https://docs.goauthentik.io/docs/property-mappings/)
- [NextAuth.js with Authentik](https://next-auth.js.org/providers/authentik)

## 🎉 Fertig!

Nach erfolgreichem Import und Test hast du eine vollständig konfigurierte Authentik-Installation für Grünerator mit:

- ✅ Benutzerregistrierung und -anmeldung
- ✅ E-Mail-Verifikation
- ✅ Sichere Passwort-Richtlinien
- ✅ Automatische Gruppenzuordnung
- ✅ Umfassende OIDC-Claims
- ✅ Logout und Session-Management
- ✅ Account-Löschung

Die Konfiguration ist produktionsreif und kann sofort mit deiner Grünerator-Anwendung verwendet werden! 🚀 
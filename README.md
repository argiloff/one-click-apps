
## Repository für CapRover One Click Apps

### So erstellen Sie eine One-Click-App (ab Version v1.8.0):

Schauen Sie sich zunächst dieses einfache Beispiel an. Lesen Sie nun weiter für mehr Details:

1. Finden oder erstellen Sie eine `docker-compose`-Datei für die gewünschte App.
2. Fügen Sie `captainVersion: 4` ganz oben in der YAML-Datei hinzu.
3. Fügen Sie diesen Abschnitt am Ende der YAML-Datei hinzu:

```yaml
caproverOneClickApp:
    variables:
        - id: '$$cap_myapp_version'
          label: Tolle App-Version
          defaultValue: '1.2.3'
          description: Sehen Sie sich deren Docker-Seite für die gültigen Tags an https://hub.docker.com/r/....../tags
          validRegex: '/.{1,}/'
    instructions:
        start: |-
            Eine Beschreibung, die dem Benutzer angezeigt wird,
            wenn er die One-Click-App installiert!
            Sie kann mehrzeilig sein und weitere Details sowie möglicherweise
            spezielle Hardwareanforderungen enthalten!
        end: |-
            Eine Zusammenfassung, wenn die App bereitgestellt ist!
            Sie kann mehrzeilig sein.

            Sie kann auch die dynamischen Parameter wie
            $$cap_appname und $$cap_other_random_char enthalten.
    displayName: Die Tolle App
    isOfficial: true ## Nur wenn alle hier verwendeten Bilder offiziell oder aus einer vertrauenswürdigen Quelle stammen.
    description: Eine relativ kurze Beschreibung, weniger als 200 Zeichen.
    documentation: Dieses docker-compose stammt von example.com
```

### Variablen:
- Variablen sind mit `$$cap` vorangestellt.
- Variablen können überall im Inhalt platziert werden und werden durch die Benutzereingaben ersetzt.
- Es gibt 3 spezielle Variablen, die für alle One-Click-Apps vorab integriert sind: `$$cap_appname`, `$$cap_root_domain` und `$$cap_gen_random_hex(length)`.
- Beispiel: Wenn Ihre App Umgebungsvariablen mit dem URL-Wert der App benötigt, können Sie `$$cap_appname.$$cap_root_domain` verwenden, das in etwa zu `myappname.rootdomain.com` aufgelöst wird.
- Falls Sie ein Standardpasswort benötigen, verwenden Sie `$$cap_gen_random_hex(10)`.
- Jede benutzerdefinierte Variable muss eine `id` und ein `label` haben. Sie können auch `defaultValue`, `validRegex` und `description` haben.

**WICHTIG**: Standardmäßig müssen Felder nicht ausgefüllt werden. Wenn `validRegex` nicht gesetzt ist, kann das Feld leer bleiben und vom Benutzer ignoriert werden.

### Services:
Obwohl das von One-Click-Apps verwendete Format Docker Compose ist, werden nicht alle in der Docker-Compose-Datei definierten Parameter von CapRover verarbeitet. Nur die folgenden Parameter werden verwendet:

- image
- environment
- ports
- volumes
- depends_on
- hostname
- command
- cap_add

Andere Parameter werden derzeit von CapRover ignoriert. Wenn Sie einen bestimmten Parameter benötigen, erstellen Sie bitte ein Issue, und wir fügen ihn zur entsprechenden Liste hinzu.

Neben der Docker-Compose-Vorlage haben Services einen speziellen Unterabschnitt für CapRover namens `caproverExtra`, der service-spezifische Parameter enthält, die nur über CapRover und nicht über Docker Compose verfügbar sind. Derzeit kann dieses Feld die folgenden Variablen annehmen:

- `dockerfileLines`, ein mehrzeiliges Feld, das anstelle der `image`-Eigenschaft im Service verwendet werden kann. Sie müssen die `image`-Eigenschaft löschen, wenn Sie diesen Parameter verwenden möchten.
- `containerHttpPort` ist nützlich, wenn der zugrunde liegende Dienst einen benutzerdefinierten Port für HTTP verwendet. Falls nicht angegeben, wird der Standardwert "80" verwendet.
- `notExposeAsWebApp` kann auf "true" gesetzt werden, wenn der zugrunde liegende Dienst keine HTTP-App ist. Dies ist nützlich für Datenbanken und andere intern verwendete Dienste.
- `websocketSupport` kann auf "true" gesetzt werden, um die WebSocket-Unterstützung automatisch zu aktivieren. Wird nur in Versionen 1.12+ unterstützt.

### Icon
Fügen Sie ein App-Icon zum `logos`-Verzeichnis hinzu!

### Testen Sie Ihre One-Click-Apps
Nachdem Sie Ihre One-Click-App-YAML-Datei erstellt haben, müssen Sie sie vor dem Erstellen eines Pull Requests testen. So testen Sie es:

1. Melden Sie sich im CapRover-Dashboard an.
2. Gehen Sie zu Apps und klicken Sie auf One-Click Apps/Databases.
3. Wählen Sie >> TEMPLATE << am unteren Rand der Dropdown-Liste.
4. Kopieren und fügen Sie Ihr YAML in das Textfeld ein und klicken Sie auf NEXT.
5. Geben Sie Werte ein und stellen Sie sicher, dass alles wie erwartet funktioniert.

### Erstellen Sie Ihr eigenes One-Click-App-Repository
Sie können Ihr eigenes privates Repository erstellen. CapRover unterstützt mehrere Repositories. Sie können neue Repository-URLs zur One-Click-App-Seite hinzufügen. Das offizielle Repository, dieses hier, ist verfügbar unter https://oneclickapps.caprover.com.

Um Ihr eigenes Repository zu erstellen:

1. Forken Sie dieses Repository.
2. Löschen Sie alle vorhandenen Apps (um doppelte Apps zu vermeiden) und fügen Sie Ihre eigenen hinzu.
3. Führen Sie `npm i` aus.
4. Führen Sie `npm run validate_apps` aus.
5. Führen Sie `npm run formatter-write` aus.
6. Führen Sie `npm run build` aus.
7. Nun können Sie den statischen Inhalt im Verzeichnis `./dist` überall hosten, wo Sie möchten. Das offizielle Repository verwendet GitHub Pages, um den Inhalt zu veröffentlichen. Aktualisieren Sie `CNAME` auf Ihre eigene URL, falls Sie dies tun möchten.

### Ihr eigenes Repository auf einer CapRover-Instanz hosten
Ihr eigenes privates Repository kann auf einer CapRover-Instanz mit der neu hinzugefügten `captain-definition`-Datei gehostet werden.

Um Ihr privates Repository auf CapRover einzurichten:

1. Folgen Sie den oben genannten Schritten, um Ihr eigenes Repository zu erstellen. Aktualisieren Sie Ihren Fork vom ursprünglichen Master-Branch, wenn sich `captain-definition` nicht im Root-Verzeichnis Ihres Forks befindet.
2. Gehen Sie im CapRover-Dashboard zu Apps. Unter "Create A New App" benennen Sie es entsprechend, z. B. `caprover-apps`, lassen Sie das Kontrollkästchen "Has Persistent Data" deaktiviert und klicken Sie auf "Create New App".
3. Gehen Sie in der neuen App zu Deployment, scrollen Sie nach unten zu "Method 3: Deploy from Github/Bitbucket/Gitlab", geben Sie die Git-URL für Ihr geforktes Repository und andere angeforderte Daten ein, klicken Sie auf "Save and Update" und dann auf "Force Build".
4. Alternativ kann eine Instanz Ihres privaten Repositorys erstellt werden, indem Sie ein Tarball (.tar) des Inhalts des One-Click-Apps-Repos erstellen und unter "Method 2: Tarball" hochladen.
5. Überprüfen Sie, dass die unter HTTP-Einstellungen angezeigte Domain die "Welcome to nginx!"-Seite anzeigt.
6. Sie sollten eine weitere Domain zu dieser CapRover-Site hinzufügen und sie als Drittanbieter-Repository mit den untenstehenden Anweisungen hinzufügen können.

### Drittanbieter-One-Click-Apps
Um ein Drittanbieter-Repository hinzuzufügen:

1. Melden Sie sich im CapRover-Dashboard an.
2. Gehen Sie zu Apps und klicken Sie auf One-Click Apps/Databases und scrollen Sie nach unten.
3. Unter 3rd party repositories: kopieren Sie die URL (zum Beispiel: https://Awes0meHub.github.io/caprover-one-click-apps) und fügen Sie sie in das Textfeld ein.
4. Klicken Sie auf die Schaltfläche "Connect New Repository".

### Drittanbieter-Repositories
- Awes0meHub: GitHub-Repository: https://caproverhub.github.io/caprover-one-click-apps
- Jordan-hall: GitHub-Repository: https://oneclickapps.libertyware.io

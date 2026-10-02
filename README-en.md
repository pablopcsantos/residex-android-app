# Residex

*Leia isto em outros idiomas: [Português](README.md)*

---

Residex is an Android application for tracking medical residency selection
processes. The app brings together applications, exams, results, and official
links in a smartphone-optimized experience, with synchronized data and local
caching.

## Features

- searchable calendar with filters by Brazilian state (UF) and status;
- independent filters for applications and exams occurring within the next seven days;
- filters and sorting in a side menu, preserving screen space for results;
- followed selection processes with persistent manual ordering;
- details on applications, fees, stages, results, calls for applications, and official links;
- manual and periodic synchronization with the configured API;
- authenticated administration to add, edit, and delete selection processes;
- configurable local notifications with duplicate prevention;
- notification permission request on Android 13 or later;
- button for sending a test notification;
- light and dark themes using the Residex visual identity;
- Room cache to keep data available between synchronizations.

## Navigation

| Screen | Purpose |
| --- | --- |
| Calendar | View followed selection processes, search, filter, and sort |
| Selection Processes | Choose processes to follow and configure their manual order |
| Administration | Authenticate and manage published data |
| Settings | Configure connection, synchronization, and notifications |

The Brazilian state (UF), status, and sorting filters are located in the side
menu of the calendar screen. The icon indicates how many filters are active,
while the main screen remains focused on search and results.

## Technology and requirements

The project uses Kotlin, Jetpack Compose, Material 3, Navigation Compose,
Hilt, Retrofit/Moshi, Room, and WorkManager.

- Java 17;
- Android SDK 36;
- Android Build Tools 36.0.0;
- minimum SDK: Android 7.1, API 25;
- target SDK: API 36.

Support for modern date APIs on older devices uses core library desugaring.

## Data source and configuration

The data published by Residex comes from a Google Sheets spreadsheet stored in
Google Drive. This spreadsheet serves as the project's managed database, but
the Android application does not access it directly. A Google Apps Script
project, deployed as a web application, acts as the intermediary: it reads and
updates the spreadsheet and exposes the results to the app in JSON format.

Data flows as follows:

1. the selection processes are maintained in the `SELEÇÕES` tab of the
   spreadsheet in Google Drive;
2. Google Apps Script reads the spreadsheet rows and transforms them into JSON
   objects;
3. the application queries the public Apps Script URL using Retrofit and Moshi;
4. after a successful synchronization, the app stores the data in a Room
   database on the device;
5. the screens query this local copy, which remains available offline or when a
   synchronization fails.

In the administration area, the flow also works in the opposite direction.
After authentication, additions, changes, and deletions are sent to Apps Script
through `POST` requests. The script validates the administrative password and,
when the operation is authorized, updates the spreadsheet in Google Drive and
returns the updated list to the app. The password does not grant the
application direct access to Google Drive and is not stored on the device.

~~~text
Google Sheets (Google Drive)
             ↕
Google Apps Script (JSON API)
             ↕
Android application (Retrofit/Moshi)
             ↕
Local Room database and app screens
~~~

The application includes a default public URL for this Google Apps Script API.
It can be changed and tested on the **Settings** screen. A deployment in another
environment must point Apps Script to the correct spreadsheet and provide the
corresponding URL to the app.

The general format returned by the public read endpoint is:

~~~json
{
  "ok": true,
  "selecoes": [
    {
      "ID": "SEL-001",
      "UF": "SP",
      "SELEÇÃO": "Institution name",
      "ATIVA": "TRUE"
    }
  ],
  "generatedAt": "2026-08-19T00:00:00.000Z"
}
~~~

To create a complete instance, from importing the spreadsheet into Google
Drive to deploying Apps Script and configuring the app, see the
[Deployment Guide](docs/GUIA_DE_IMPLANTACAO.md).

## Security and privacy

- the public API URL does not contain administrative credentials;
- the administrative password is sent only in authenticated operations and is
  not persisted by the application;
- following preferences, ordering, and notification settings remain on the
  device;
- the project does not include advertising or analytics libraries;
- passwords, tokens, keystores, and environment files must not be added to the
  repository.

The material in `reference/apps-script/` contains an auxiliary password
configuration routine. The temporary constant must remain empty in versioned
code.

## Local build

The Codespace can install the SDK automatically using
`.devcontainer/devcontainer.json` and `scripts/setup-android-sdk.sh`.

For environments that are already configured, run the tasks separately to
reduce peak memory usage:

~~~bash
./gradlew --no-daemon --max-workers=1 test
./gradlew --no-daemon --max-workers=1 lintDebug
./gradlew --no-daemon --max-workers=1 assembleDebug
~~~

The development APK is generated at:

~~~text
app/build/outputs/apk/debug/app-debug.apk
~~~

## Tests and checks

Unit tests cover date and status rules, repository synchronization,
notifications, and selection-process ordering. The `test` task runs the debug
and release variants.

Before submitting changes, it is recommended to run:

~~~bash
./gradlew --no-daemon --max-workers=1 test lintDebug assembleDebug
~~~

In low-memory environments, prefer the three separate commands shown in the
previous section. Instrumented tests on a device are not yet part of the
repository.

## Relevant structure

~~~text
app/src/main/java/com/pablopcsantos/residex/
├── navigation/             Main application navigation
├── residency/data/         API, Room, preferences, and repositories
├── residency/domain/       Selection models and rules
├── residency/notification/ Channels, preferences, and notification rules
├── residency/ui/           Product screens and ViewModels
├── residency/work/         Periodic synchronization and notifications
└── ui/theme/               Material 3 palette, typography, and theme

app/src/test/               Unit tests
reference/                  Web sources and documents used as reference
~~~

The `reference/` directory is not packaged in the APK. It contains the
backend Apps Script and a reference data spreadsheet.

## Android identity

- display name: Residex;
- namespace and application ID: `com.pablopcsantos.residex`;
- Application class: `ResidexApp`;
- Android theme: `Theme.Residex`;
- current project version: 3.2.4 (versionCode 3).

---

## 👤 Authorship and development

Residex is a native Android application independently developed by **Pablo Phillipe Cândido dos Santos**, intended to help candidates track medical residency selection processes. It brings together calendars, deadlines, stages, calls for applications, and information links, with search, filters, configurable notifications, and local availability of synchronized data.

Generative artificial intelligence tools were used as auxiliary resources during development, while responsibility for the project's conception, implementation, integration, and verification remained with the author.

Lattes CV: [http://lattes.cnpq.br/9500873674712528](http://lattes.cnpq.br/9500873674712528)

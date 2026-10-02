# Betaling med Stripe
Velkommen til dagens øvelse, hvor vi skal tage imod betalinger i vores app. Vi bygger en lille kantine-shop, hvor du vælger en vare, trykker Betal og indtaster et kort. Vi bruger Stripes **testmode**, så der bliver aldrig flyttet rigtige penge.

Du får bidder af koden og instruktioner til, hvordan du selv færdiggør den.

## Du kan læse mere her
- Stripe Payment Links: https://docs.stripe.com/payment-links
- Stripes testkort: https://docs.stripe.com/testing
- expo-web-browser: https://docs.expo.dev/versions/latest/sdk/webbrowser/
- expo-linking: https://docs.expo.dev/versions/latest/sdk/linking/

## Sådan hænger det sammen

1. Du laver et betalingslink i Stripes dashboard. Det er en helt almindelig webadresse.
2. Appen åbner linket i en browser oven på appen.
3. Brugeren betaler med testkortet og lukker vinduet.
4. Appen mærker, at vinduet er lukket, og viser en kvittering.

Vi skal altså **ikke** have en server, og vi skal **ikke** have nogen hemmelige nøgler ned i appen.

<br></br>

## Del 1 - Opret en Stripe-konto i testmode

1. Gå til https://dashboard.stripe.com/register og opret en konto. Du skal **ikke** udfylde noget om firma, bank eller CVR. Du kan bruge testmode med det samme.

2. Når du er inde, så tjek at der står **Test mode** eller **Sandbox** øverst. Hvis der ikke gør, så slå det til. Alt hvad vi laver i dag, skal ligge i testmode.

3. Find menupunktet **Product catalogue** i venstre side, og tryk **Add a product**:
   - Name: `Kaffe`
   - Amount: `25.00`
   - Currency: `DKK`
   - Vælg **One-off** (altså ikke abonnement)
   - Tryk **Add product**

4. Gentag, så du har tre varer i alt:
   - `Kaffe` til 25 kr.
   - `Croissant` til 30 kr.
   - `Frokost` til 75 kr.

<br></br>

## Del 2 - Lav tre betalingslinks

1. Gå til **Payment links** i menuen, og tryk **Create a payment link**.

2. Vælg produktet `Kaffe` på listen.

3. Rør **ikke** indstillingen om **After payment**. Lad den stå på Stripes egen kvitteringsside. Så får brugeren en "Thanks for your payment"-side, og derefter lukker han selv vinduet.

4. Tryk **Create link**, og kopier linket. Det ser sådan her ud:
   ```
   https://buy.stripe.com/test_7sYaEX1234abcd5678
   ```
   *Læg mærke til at der står `test_` i linket. Gør der ikke det, er du ikke i testmode, og så skal du stoppe og starte forfra.*

5. Gem linket et sted, hvor du kan finde det igen. Fx i en note eller direkte i din kode.

6. **Gentag punkt 1 til 5 for `Croissant` og `Frokost`.** Du skal ende med tre links.

<br></br>

## Del 3 - Opret projekt & mappestruktur

1. Åbn en terminal og navigér til den mappe, du gerne vil gemme dit projekt i, med `cd sti-til-din-mappe-her`

2. Opret et nyt React Native projekt:
    ```
    npx create-expo-app@latest BetalingsApp --template blank --no-agents-md
    ```
    *Kopier hele linjen som den står.*

3. Navigér ind i projektet:
    ```
    cd BetalingsApp
    ```

4. Installer de pakker, vi skal bruge:
    ```
    npx expo install expo-web-browser expo-linking @react-navigation/native @react-navigation/stack react-native-screens react-native-safe-area-context react-native-gesture-handler
    ```
    *Brug `npx expo install` og ikke `npm install`. Expo sørger for, at du får de versioner, der passer til Expo Go på din telefon.*

5. Opret mapperne og filerne:
    ```
    mkdir components screens services styles
    touch components/StackNavigator.js screens/ShopScreen.js screens/ReceiptScreen.js services/ProductData.js services/Payment.js styles/GlobalStyles.js
    ```

<br></br>

## Del 4 - Giv appen en adresse i app.json

For at browseren kan finde tilbage til din app, skal appen have sin egen adresse. Det kaldes et `scheme`.

1. Åbn `app.json`, og tilføj linjen `"scheme": "betalingsapp"` inde i `expo`-blokken:
    ```json
    {
      "expo": {
        "name": "BetalingsApp",
        "slug": "BetalingsApp",
        "scheme": "betalingsapp",
        "version": "1.0.0",
        ...
      }
    }
    ```
    *Pas på kommaerne. Hver linje undtagen den sidste i blokken skal slutte med et komma.*

2. Hvis din app allerede kører, skal du stoppe den med `Ctrl + C` og starte den igen. `scheme` bliver kun læst, når appen starter.

<br></br>

## Del 5 - services/ProductData.js

Her gemmer vi vores varer. Hver vare har et navn, en pris og det betalingslink, du lavede i Del 2.

1. Åbn `services/ProductData.js`:
    ```javascript
    const products = [
      { id: 1, navn: "Kaffe", pris: 25, link: "DIT-KAFFE-LINK-HER" },
      // Tilføj Croissant og Frokost på samme måde

    ];

    export default products;
    ```

2. **Opgave**:
   - Indsæt dine tre betalingslinks fra Del 2.
   - Tilføj `Croissant` til 30 kr. og `Frokost` til 75 kr., så der er tre varer i listen.
   - Husk at hver vare skal have sit **eget** `id`.

<br></br>

## Del 6 - services/Payment.js

Det er her, det sjove sker. Vi åbner betalingslinket i en browser og venter på, at brugeren er færdig.

1. Åbn `services/Payment.js`:
    ```javascript
    import * as WebBrowser from "expo-web-browser";
    import * as Linking from "expo-linking";

    export default async function betal(betalingsLink) {
      // 1. Lav den adresse, browseren skal sende brugeren tilbage til

      // 2. Åbn betalingslinket med openAuthSessionAsync

      // 3. Returnér resultatet
    }
    ```

2. **Opgave**:
   - Lav retur-adressen med `Linking.createURL("kvittering")`, og gem den i en variabel.
        - HINT: `const returUrl = Linking.createURL("kvittering")`
   - Åbn betalingslinket med `WebBrowser.openAuthSessionAsync()`. Den skal have to ting med: betalingslinket og din retur-adresse.
        - HINT: `const resultat = await WebBrowser.openAuthSessionAsync(betalingsLink, returUrl)`
   - Returnér `resultat`.

3. **Hvad får vi tilbage?**
   `openAuthSessionAsync` venter, indtil brugeren lukker browseren, og giver os så et objekt med en `type`, fx:
    ```javascript
    { type: "cancel" }    // brugeren lukkede vinduet
    ```
   Det er altså lukningen af vinduet, vi bruger som signal om, at brugeren er færdig.

   *Brug `openAuthSessionAsync` og ikke `openBrowserAsync`. På Android vender `openBrowserAsync` tilbage med det samme, allerede før brugeren har betalt. `openAuthSessionAsync` venter på begge platforme.*

<br></br>

## Del 7 - styles/GlobalStyles.js

Al vores styling skal ligge i én fil, så koden bliver nem at læse.

1. Åbn `styles/GlobalStyles.js`, og indsæt:
    ```javascript
    import { StyleSheet } from "react-native";

    const GlobalStyles = StyleSheet.create({
      container: { flex: 1, backgroundColor: "#fff", padding: 20, paddingTop: 60 },
      overskrift: { fontSize: 28, fontWeight: "bold", marginBottom: 20 },
      vare: {
        flexDirection: "row",
        justifyContent: "space-between",
        alignItems: "center",
        padding: 16,
        backgroundColor: "#F5F5F5",
        borderRadius: 10,
        marginBottom: 10,
      },
      vareNavn: { fontSize: 18 },
      varePris: { fontSize: 18, fontWeight: "bold" },
      knap: {
        backgroundColor: "#635BFF",
        padding: 16,
        borderRadius: 100,
        alignItems: "center",
        marginTop: 20,
      },
      knapTekst: { color: "#fff", fontSize: 16, fontWeight: "bold" },
      status: { marginTop: 20, fontSize: 16, textAlign: "center" },
      kvitteringBoks: { alignItems: "center", justifyContent: "center", flex: 1 },
      kvitteringTitel: { fontSize: 24, fontWeight: "bold", marginBottom: 10 },
      kvitteringNote: { marginTop: 20, paddingHorizontal: 40, textAlign: "center", color: "#666" },
    });

    export default GlobalStyles;
    ```

<br></br>

## Del 8 - screens/ShopScreen.js

Her viser vi varerne i en liste, lader brugeren vælge én, og sender ham videre til betaling.

1. Åbn `screens/ShopScreen.js`:
    ```javascript
    import { useState } from "react";
    import { View, Text, FlatList, TouchableOpacity } from "react-native";
    import products from "../services/ProductData";
    import betal from "../services/Payment";
    import GlobalStyles from "../styles/GlobalStyles";

    export default function ShopScreen({ navigation }) {
      const [valgt, setValgt] = useState(products[0]);
      const [status, setStatus] = useState("");

      const paaBetal = async () => {
        // Her skal betalingen startes

      };

      return (
        <View style={GlobalStyles.container}>
          <Text style={GlobalStyles.overskrift}>Kantinen</Text>

          <FlatList
            data={products}
            keyExtractor={(item) => String(item.id)}
            renderItem={({ item }) => (
              <TouchableOpacity
                style={[
                  GlobalStyles.vare,
                  item.id === valgt.id && { borderWidth: 2, borderColor: "#635BFF" },
                ]}
                onPress={() => setValgt(item)}
              >
                <Text style={GlobalStyles.vareNavn}>{item.navn}</Text>
                <Text style={GlobalStyles.varePris}>{item.pris} kr.</Text>
              </TouchableOpacity>
            )}
          />

          {/* Opret en TouchableOpacity, der kalder paaBetal */}

          <Text style={GlobalStyles.status}>{status}</Text>
        </View>
      );
    }
    ```

2. **Opgave - færdiggør `paaBetal`**:
   - Sæt status til `"Åbner betaling ..."`, så brugeren kan se, at der sker noget.
   - Kald `betal()` med linket fra den valgte vare, og gem svaret i en variabel.
        - HINT: `const resultat = await betal(valgt.link)`
        - Husk `await`. Uden `await` får du et Promise i stedet for et svar, og så sker der ingenting.
   - Når browseren er lukket, så sæt status til en tekst, der viser `resultat.type`. Så kan du selv se, hvad der kom tilbage.
   - Navigér til kvitteringen, og send varens navn med:
        - HINT: `navigation.navigate("kvittering", { vare: valgt.navn })`
   - Pak det hele i `try` og `catch`, så en fejl ikke vælter appen.

3. **Opgave - lav knappen**:
   - Opret en `TouchableOpacity` med `style={GlobalStyles.knap}` og `onPress={paaBetal}`.
   - Inde i den skal der være en `Text` med `style={GlobalStyles.knapTekst}`, der viser prisen og navnet på den valgte vare, fx "Betal 25 kr. for Kaffe".

<br></br>

## Del 9 - screens/ReceiptScreen.js

Kvitteringen skal vise, hvad brugeren købte. Navnet kommer med i navigationen, så vi henter det med `route.params`.

1. Åbn `screens/ReceiptScreen.js`:
    ```javascript
    import { View, Text } from "react-native";
    import GlobalStyles from "../styles/GlobalStyles";

    export default function ReceiptScreen({ route }) {
      // Hent varens navn ud af route.params

      return (
        // Vis en overskrift og en tekst med varens navn
      );
    }
    ```

2. **Opgave**:
   - Hent varens navn ud af `route.params`.
        - HINT: `const vare = route.params?.vare ?? "din vare"`
        - `??` betyder "hvis der ikke er noget, så brug dette i stedet". Det gør, at skærmen ikke går ned, hvis den åbnes uden en vare.
   - Lav en `View` med `style={GlobalStyles.kvitteringBoks}`.
   - Indeni skal der være en `Text` med `style={GlobalStyles.kvitteringTitel}`, der siger "Tak for købet", og en `Text`, der siger "Du har betalt for {vare}."
   - Tilføj en `Text` med `style={GlobalStyles.kvitteringNote}`, der siger "Betalingen står nu i Stripes dashboard under Payments."

<br></br>

## Del 10 - components/StackNavigator.js

Nu skal de to skærme kobles sammen.

1. Åbn `components/StackNavigator.js`:
    ```javascript
    import { createStackNavigator } from "@react-navigation/stack";
    import { NavigationContainer } from "@react-navigation/native";
    import ShopScreen from "../screens/ShopScreen";
    import ReceiptScreen from "../screens/ReceiptScreen";

    const Stack = createStackNavigator();

    export default function ShopNavigation() {
      return (
        <NavigationContainer>
          <Stack.Navigator>
            {/* Opret to Stack.Screen: shop og kvittering */}

          </Stack.Navigator>
        </NavigationContainer>
      );
    }
    ```

2. **Opgave**:
   - Opret en `Stack.Screen` med `name="shop"` og `component={ShopScreen}`. Skjul headeren med `options={{ headerShown: false }}`.
   - Opret en `Stack.Screen` med `name="kvittering"` og `component={ReceiptScreen}`.
   - Navnene skal stå **præcis** som de gør, når du navigerer i `ShopScreen`. Skriver du `Kvittering` med stort her og `kvittering` med lille i navigationen, så går appen ned.

<br></br>

## Del 11 - App.js

1. Åbn `App.js`, slet alt indholdet, og indsæt:
    ```javascript
    import ShopNavigation from "./components/StackNavigator";

    export default function App() {
      return <ShopNavigation />;
    }
    ```

<br></br>

## Del 12 - Test din app

1. Start appen:
    ```
    npx expo start
    ```

2. Scan QR-koden med Expo Go, vælg en vare og tryk Betal. Stripes betalingsside skal åbne oven på appen.

3. Betal med Stripes testkort:
   - Kortnummer: `4242 4242 4242 4242`
   - Udløbsdato: en dato ude i fremtiden, fx `12 / 34`
   - CVC: tre vilkårlige cifre, fx `123`
   - Postnummer: fire vilkårlige cifre, fx `2000`

   *Brug kun dette kort. Skriv aldrig dit rigtige kortnummer ind i en øvelse.*

4. Når Stripe siger tak for betalingen, så luk browser-vinduet med **Luk** eller **Done** øverst. Så skal du se kvitteringen i appen.

5. Gå tilbage til Stripes dashboard og find **Payments** i menuen. Din betaling står på listen med teksten **Succeeded** og en note om, at den er lavet i testmode. Det er her, man ser, om pengene faktisk er kommet ind.

<br></br>

## Hvis det ikke virker

| Problem | Løsning |
| --- | --- |
| Browseren åbner slet ikke | Står der et rigtigt link i `ProductData.js`? Har du husket `await` i `paaBetal`? |
| Kvitteringen kommer med det samme, før du har betalt | Du bruger `openBrowserAsync`. Skift til `openAuthSessionAsync` |
| Appen reagerer ikke, når du lukker browseren | Har du `"scheme": "betalingsapp"` i `app.json`? Har du genstartet Expo efter du tilføjede den? |
| "Cannot read property type of undefined" | Du mangler `await` foran `betal()` |
| Appen går ned når du navigerer | Hedder skærmen præcis det samme i `navigate()` og i `Stack.Screen`? |
| Stripe siger at linket ikke findes | Står der `test_` i linket? Er du i testmode i dashboardet? |

<br></br>

## Ekstra / Udfordring

1. Tilføj en kurv, så man kan vælge flere varer, og vis totalen på knappen.
2. Vis en `ActivityIndicator`, mens browseren er åben, så brugeren kan se, at der arbejdes.
3. Gem de køb, der er gået igennem, med `AsyncStorage`, og vis dem på en tredje skærm som en købshistorik.

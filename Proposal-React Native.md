# Native Modules Support on React Native

## Introduction
The purpose of this document is to give an overview of what needs to be done on React Native in order to support building widgets natively on iOS and Android.

## Native Modules

There will be three native modules that act as the middle man for data communication between React Native and the native code, as follows:
* `AppPreferencesService`
* `WatchlistService`
* `UserService`

### AppPreferencesService
This service manages app related preferences such as preferred language, currency, theme and so on. Currency is crucial for API calls that require a `vs_currency` query parameter, this ensures that the currency shown in the widgets is consistent with the preferred currency in the app. Language will be used for localization purpose.

The public APIs in the service are as follows:
- `setCurrency(currency: String)`
- `setLanguage(language: String)`

#### Sample usages of the module:

```
import { NativeModules } from 'react-native';

class App extends Component {
    componentDidMount() {
        const currency = // Get currency from AsyncStorage.
        const language = // Get language from AsyncStorage.
        
        // Assign the currency and language values whenver the app launches to keep
        // the data up to date.
	NativeModules.AppPreferencesService.setCurrency(currency)
	NativeModules.AppPreferencesService.setLanguage(language)
    }
}

class Settings extends Component {
    changeCurrency = (currency) => {
        // Reassign the currency value whenever user changes the 
        // currency within the app.
        NativeModules.AppPreferencesService.setCurrency(currency)
    }
    
    changeLanguage = (language) => {
        // Reassign the language value whenever user changes the 
        // language within the app.
        NativeModules.AppPreferencesService.setLanguage(currency)
    }
}
```

### WatchlistService
This service manages watched coins, it's the responsibility of the React Native app to manage the watchlist using this module.

The public APIs in the service are as follows:
- `getWatchlist(): String<Optional>`
- `setWatchlist(jsonReprensentation: String<Optional>)`

The deliberate use of `string` type for watchlist makes accessing data between different platforms possible. That also means the React Native side has to convert the watchlist to JSON representation for data storing and vice versa.

#### Sample usages of the module:

```
import { NativeModules } from 'react-native';

class WidgetPreferences extends Component {
    render() {
        // This might no be the best place, it's just for demonstration.
        // Read data from the module.
        const json = NativeModules.WatchlistService.getWatchlist()
        
        // Parse JSON back to arrray.
        const coins = JSON.parse(json)

        // Render UI with parsed data.
        ...
    }

    setWatchlist = (coins: [Coin]) => {
        // Note: Watchlist management UI and logic lie in the React Native app.
        // Convert array to JSON.
        const json = JSON.stringify(coins)
        NativeModules.WatchlistService.setWatchlist(json)
    }
}
```

### UserService
This service manages current user logged in state, which is used by widgets to populate appropriate placeholders. The API in the service is `setIsUserLoggedIn(value: Boolean)`.

#### Sample usages of the module:
```
import { NativeModules } from 'react-native';

class App extends Component {

    // Sample observer pattern.
    user.onUserChanged { user =>
        NativeModules.UserService.setIsUserLoggedIn(user != null)
    }
}
```


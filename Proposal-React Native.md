# Native Modules Support on React Native

## Introduction
The purpose of this document is to give an overview of what needs to be done on React Native in order to support building widgets natively on iOS and Android.

## Native Modules

There will be two native modules that act as the middle man for data communication between React Native and the native code, as follows:
* `AppPreferencesService`
* `WatchlistService`

### AppPreferencesService
This service manages app related preferences such as preferred language, currency, theme and so on. Currency is crucial for API calls that require a `vs_currency` query parameter, this ensures that the currency shown in the widgets is consistent with the preferred currency in the app. Language will be used for localization purpose.

The public methods of `AppPreferencesService` are `setCurrency(currency: String)` and `setLanguage(language: String)`. It's the responsibility of the React Native app to assign currency and language values using this module. 

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
        NativeModules.AppPreferencesService.setCurrency(currency)
    }
}
```

### WatchlistService
This service manages watched coins, it's the responsibility of the React Native app to manage the watchlist using this module.

The public APIs in the service are as  follows:
- `getWatchlist(): String<Optional>`
- `setWatchlist(jsonReprensentation: String<Optional>)`

The deliberate use of `string` type for watchlist makes accessing data between different platforms possible. That also means the React Native side has to convert the watchlist to JSON representation for data storing and vice versa.

#### State
The availability of the watchlist represents different types of states:
- Null: User is not logged in to the app.
- Empty: User is logged in to the app but has empty watchlist.
- Non-Empty: User is logged in to the app with watched coins.

An empty watchlist basically means an empty JSON, `{ }`. The widgets will then present appropriate placeholder based on the state of the data.  The sample usage section below describes how these states can be managed properly in React Native.


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

class Settings extends Component {
    logOut = () => {
        // Set watchlist to null, which indicates that the user
        // has no longer signed in.
        NativeModules.WatchlistService.setWatchlist(null)
    }
}
```

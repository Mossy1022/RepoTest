# TrellixUserRegistration

This library was generated with [Angular CLI](https://github.com/angular/angular-cli) version 7.2.0.

# Setup

This library opens the cloud portal for universal registration among apps.  To use, this module must be imported within the parent component of the application (typically that is the app component, but may vary).

The component should use the TrellixUserRegistrationStateService to subscribe to the 3 states and respond appropriately:

- getConnectionStateModel()
  
  Lets you know if the website being routed to does not respond or emits an error.  Does not check if the current device has internet connection.

- getLoginCheckedStateModel()

  Lets you know that the LoggedInStateModel has been updated from it's default values after checking local storage and ensuring the logout expiration time hasn't been exceeded since last login

- getLoggedInStateModel()

  Lets you know if the user has logged in.
 
 # Trellix Cordova Network Module
 
For checking if the device is connected to internet, the TrellixCordovaNetworkStateService from the trellix-cordova-network library is used in conjunction.  The parent component should subscribe to the getNetworkStateSub method of the service. Below is an example of how the component handles a change in connection state.

```
private _checkInternetConnection() : void {
    this._ngZone.run(() => {
      this._trellixNetworkService.getNetworkStateSub().pipe(takeWhile(() => this._compActive)).subscribe((connection : string) => {
        console.log('connection!! ', connection);
        if (connection !== CONN_STATUS.ONLINE && connection !== CONN_STATUS.EMPTY_STRING) {
          // Update markup and config of try again component to display
          this.cordovaMarkup = {
            line1: this.markup.descriptions.no_network_line_1,
            line2: this.markup.descriptions.no_network_line_2
          };
          this.tryAgainConfig = {
            showActionButton: false
          };
          this.noInternet = true;
        } else {
          this.noInternet = false;
        }
      });
    });
  }
  
  ```

# Trellix Try Again

If you noted in the above code, there is a markup and configuration that is set if there is a connection error.  
In conjunction with the other two modules, the Trellix Try Again module is used to display errors in connection and must also be imported to the parent app if you wish to use it.

The above code updates the markup object sent into the trellix-try-again component that is used to display connection errors, with or without a try again button depending on the showActionButton configuration.  For clarity, trellix try again has 2 inputs:

- Configuration
- Markup Object

As well as an output for btn-clicked.  In this case, the retry button.
 
 # Example
 
 Below is an example of a function that checks the login/connection state to manage the app:
 
 ```
private _checkLogin() : void {
   
   // Sub to connection state if website doesn't respond. Defaults to true.  
    this._trellixRegService.getConnectionStateModel().pipe(takeWhile(() => this._compActive)).subscribe((connection : ConnectionStateModel) => {
       
       // NgZone is sometimes needed to ensure it's running in the main thread
      this._ngZone.run(() => {
        
        // If there is internet, but no connection to the server - show server try-again page
        if (!connection.connected  && !this.noInternet) {
          
          // Update markup and config of try again component to display.  Since a problem with
          // The server and not the internet connection, no option to try again
          this.cordovaMarkup = {
            line1: this.markup.descriptions.not_responding_line_1,
            line2: this.markup.descriptions.not_responding_line_2,
            button : this.markup.labels.try_again
          };
          this.tryAgainConfig = {
            showActionButton: true
          };
          
          // This variable is checked in the html markup for whether or not to display the try again componenent instead of user registration
          this.connectionError = true;
        } else {
          this.connectionError = false;
        }
      });
    });

    // Sub to loginchecked state to determine when its ok to get login state after retrieving from local storage
    this._trellixRegService.getLoginCheckedStateModel().pipe(takeWhile(() => this._compActive)).subscribe((loginCheck : LoginCheckedStateModel) => {
      this._ngZone.run(() => {
        this.loginChecked = loginCheck.checked;
      });
    });

    // Sub to loggedInstate
    this._trellixRegService.getLoggedInStateModel().pipe(takeWhile(() => this._compActive)).subscribe((login : LoggedInStateModel) => {
      // Only check logged in state after it has been checked in local storage
      this._ngZone.run(() => {
        if (this.loginChecked) {
          if (login.isLoggedIn) {
            // Setting to true will show mdns
            this.isLoggedIn = true;
          } else {
            this.isLoggedIn = false;
          }
        }
      });
    });
  }
```

HTML Code

```
<div *ngIf="isLoggedIn">
  <eaton-auth-mdns *ngIf="isDevice" (doLoginActions)="submitLogin($event)"
                   (demoModeChanged)="demoModeChanged($event)" [requestSent]="requestSent" (side-action)="processSideNavAction($event)"></eaton-auth-mdns>
  <eaton-auth-authorization *ngIf="!isDevice"></eaton-auth-authorization>
</div>
<div *ngIf="!isLoggedIn && !connectionError && loginChecked">
  <trellix-user-registration></trellix-user-registration>
</div>
<div *ngIf="!isLoggedIn && (connectionError || noInternet) && loginChecked">
  <trellix-try-again [markup]="cordovaMarkup" [config]="tryAgainConfig" (btn-pressed)="retryConnecting()" ></trellix-try-again>
 </div>
 ```

## Code scaffolding

Run `ng generate component component-name --project trellix-user-registration` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module --project trellix-user-registration`.
> Note: Don't forget to add `--project trellix-user-registration` or else it will be added to the default project in your `angular.json` file. 

## Build

Run `ng build trellix-user-registration` to build the project. The build artifacts will be stored in the `dist/` directory.

## Publishing

After building your library with `ng build trellix-user-registration`, go to the dist folder `cd dist/trellix-user-registration` and run `npm publish`.

## Running unit tests

Run `ng test trellix-user-registration` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md).

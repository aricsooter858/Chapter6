# Chapter 6 from the Wilken book with challenges

Code help from https://github.com/ahuimanu/cidm4385-2016sp-ionic-basics

This code is from Chapter 6 as well as the challenges at the end of the chapter

index.html

```

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
    <title></title>

    <link href="lib/ionic/css/ionic.css" rel="stylesheet">
    <link href="css/style.css" rel="stylesheet">

    <!-- IF using Sass (run gulp sass first), then uncomment below and remove the CSS includes above
    <link href="css/ionic.app.css" rel="stylesheet">
    -->

    <!-- ionic/angularjs js -->
    <script src="lib/ionic/js/ionic.bundle.js"></script>
    <script src="lib/moment/moment.js"></script>
    <script src="lib/moment-timezone/builds/moment-timezone-with-data.js"></script>

    <!-- cordova script (this will be a 404 during development) -->
    <script src="cordova.js"></script>

    <!-- your app's js -->
    <script src="js/app.js"></script>
    <script src="views/search/search.js"></script>
    <script src="views/settings/settings.js"></script>
    <script src="views/weather/weather.js"></script>
  </head>
  <body ng-app="App">
    <ion-side-menus>
      <ion-side-menu-content>
        <ion-nav-bar class="bar-positive">
          <ion-nav-buttons side="left">
            <button class="button button-clear" menu-toggle="left">
              <span class="icon ion-navicon"></span>
            </button>
          </ion-nav-buttons>
        </ion-nav-bar>
        <ion-nav-view></ion-nav-view>
      </ion-side-menu-content>
      <ion-side-menu side="left" ng-controller="LeftMenuController">
        <ion-header-bar class="bar-dark">
          <h1 class="title">My Weather</h1>
        </ion-header-bar>
        <ion-content>
          <ion-list>
            <ion-item class="item-icon-left" ui-sref="search" menu-close><span class="icon ion-search"></span> Find a City</ion-item>
            <ion-item class="item-icon-left" ui-sref="settings" menu-close><span class="icon ion-ios-cog"></span> Settings</ion-item>
            <ion-item class="item-divider">Favorites</ion-item>
            <ion-item class="item-icon-left" ui-sref="weather({city: location.city, lat: location.lat, lng: location.lng})" menu-close ng-repeat="location in locations"><span class="icon ion-ios-location"></span> {{location.city}}</ion-item>
          </ion-list>
        </ion-content>
      </ion-side-menu>
    </ion-side-menus>
  </body>
</html>



```

app.js

```

angular.module('App', ['ionic'])
.factory('Locations', function ($ionicPopup) {
  var Locations = {
    data: [{
      city: 'Amarillo, TX, USA',
      lat: 35.2019468,
      lng: -101.8749805
    }],
    getIndex: function (item) {
      var index = -1;
      angular.forEach(Locations.data, function (location, i) {
        if (item.lat == location.lat && item.lng == location.lng) {
          index = i;
        }
      });
      return index;
    },
    toggle: function (item) {
      var index = Locations.getIndex(item);
      if (index >= 0) {
        $ionicPopup.confirm({
          title: 'Are you sure?',
          template: 'This will remove ' + Locations.data[index].city
        }).then(function (res) {
          if (res) {
            Locations.data.splice(index, 1);
          }
        });
      } else {
        Locations.data.push(item);
        $ionicPopup.alert({
          title: 'Location saved'
        });
      }
    },
    primary: function (item) {
      var index = Locations.getIndex(item);
      if (index >= 0) {
        Locations.data.splice(index, 1);
        Locations.data.splice(0, 0, item);
      } else {
        Locations.data.unshift(item);
      }
    }
  };

  return Locations;
})

.config(function ($stateProvider, $urlRouterProvider, LocationsProvider) {

  $stateProvider
    .state('search', {
      url: '/search',
      controller: 'SearchController',
      templateUrl: 'views/search/search.html'
    })
    .state('settings', {
      url: '/settings',
      controller: 'SettingsController',
      templateUrl: 'views/settings/settings.html'
    })
    .state('weather', {
      url: '/weather/:city/:lat/:lng',
      controller: 'WeatherController',
      templateUrl: 'views/weather/weather.html'
    });

 console.log(LocationsProvider.data);
 
 if(LocationsProvider.$get().data.length > 0){
    $urlRouterProvider.otherwise('/weather/' + 
                                 LocationsProvider.$get().data[0].city + '/' +
                                 LocationsProvider.$get().data[0].lat + '/' + 
                                 LocationsProvider.$get().data[0].lng); 
  }else{
    $urlRouterProvider.otherwise('/search'); 
  }

})


.run(function($ionicPlatform) {
  $ionicPlatform.ready(function() {
    if(window.cordova && window.cordova.plugins.Keyboard) {
      cordova.plugins.Keyboard.hideKeyboardAccessoryBar(true);
    }
    if(window.StatusBar) {
      StatusBar.styleDefault();
    }
  });
})

.controller('LeftMenuController', function ($scope, Locations) {
  $scope.locations = Locations.data;
})

.filter('timezone', function () {
  return function (input, timezone) {
    if (input && timezone) {
      var time = moment.tz(input * 1000, timezone);
      return time.format('LT');
    }
    return '';
  };
})

.filter('chance', function () {
  return function (chance) {
    if (chance) {
      var value = Math.round(chance * 10);
      return value * 10;
    }
    return 0;
  };
})

.filter('icons', function () {
  var map = {
    'clear-day': 'ion-ios-sunny',
    'clear-night': 'ion-ios-moon',
    rain: 'ion-ios-rainy',
    snow: 'ion-ios-snowy',
    sleet: 'ion-ios-rainy',
    wind: 'ion-ios-flag',
    fog: 'ion-ios-cloud',
    cloudy: 'ion-ios-cloudy',
    'partly-cloudy-day': 'ion-ios-partlysunny',
    'partly-cloudy-night': 'ion-ios-cloudy-night'
  };
  return function (icon) {
    return map[icon] || '';
  }
})

.factory('Settings', function () {
  var Settings = {
    units: 'us',
    days: 8
  };
  return Settings;
})






```

search.html

```

<ion-view view-title="Find Locations">
  <ion-content>
    <div class="list">
      <div class="item item-input-inset">
        <label class="item-input-wrapper">
          <input type="search" ng-model="model.term" placeholder="Search for a location">
        </label>
        <button class="button button-small button-positive" ng-click="search()">Submit</button>
      </div>
      <div class="item" ng-repeat="result in results" ui-sref="weather({city: result.formatted_address, lat: result.geometry.location.lat, lng: result.geometry.location.lng})">{{result.formatted_address}}</div>
    </div>
  </ion-content>
</ion-view>



```

search.js

```

angular.module('App')
.controller('SearchController', function ($scope, $http) {
  $scope.model = {term: ''};

  $scope.search = function () {
    $http.get('https://maps.googleapis.com/maps/api/geocode/json', {params: {address: $scope.model.term}}).success(function (response) {
      $scope.results = response.results;
    });
  };
});



```

settings.html

```

<ion-view view-title="Settings">
  <ion-content>
    <ion-list>
      <ion-item class="item-divider">Units</ion-item>
      <ion-radio ng-model="settings.units" ng-value="'us'">Imperial (Fahrenheit)</ion-radio>
      <ion-radio ng-model="settings.units" ng-value="'si'">Metric (Celsius)</ion-radio>
      <div class="item item-divider">Days in forecast <span class="badge badge-dark">{{settings.days - 1}}</span></div>
      <div class="item range range-positive">
        2 <input type="range" name="days" ng-model="settings.days" min="2" max="8" value="8"> 8
      </div>
      <div class="item item-button-right">Favorites <button class="button button-small" ng-click="canDelete = !canDelete">{{canDelete ? 'Done' : 'Edit'}}</button></div>
    </ion-list>
    <ion-list show-delete="canDelete">
      <ion-item ng-repeat="location in locations">
        <ion-delete-button class="ion-minus-circled" ng-click="remove($index)"></ion-delete-button>
        {{location.city}}
      </ion-item>
    </ion-list>
    <p class="padding">Weather data powered by <a href="https://developer.forecast.io/docs/v2">Forecast.io</a> and geocoding powered by <a href="https://developers.google.com/maps/documentation/geocoding/">Google</a>.</p>
  </ion-content>
</ion-view>



```

settings.js

```

angular.module('App')
.controller('SettingsController', function ($scope, Settings, Locations) {
  $scope.settings = Settings;
  $scope.locations = Locations.data;
  $scope.canDelete = false;

  $scope.remove = function (index) {
    Locations.toggle(Locations.data[index]);
  };
});



```

weather.html

```

<ion-view view-title="{{params.city}}">
  <ion-nav-buttons side="left">
    <button class="button button-clear" menu-toggle="left"><span class="icon ion-navicon"></span></button>
  </ion-nav-buttons>
  <ion-nav-buttons side="right">
    <button class="button button-icon" ng-click="showOptions()"><span class="icon ion-more"></span></button>
  </ion-nav-buttons>
  <ion-content>
    <ion-scroll direction="y" paging="true" ng-style="{width: getWidth(), height: getHeight()}">
      <ion-refresher
        pulling-text="Obtaining weather"
        on-refresh="doRefresh()">
      </ion-refresher>
      <div ng-style="{height: getTotalHeight()}">
        <div class="scroll-page center" ng-style="{width: getWidth(), height: getHeight()}">
          <div class="bar bar-dark">
            <h1 class="title">Current Conditions</h1>
          </div>
          <div class="has-header">
            <h2 class="primary">{{forecast.currently.temperature | number:0}}&deg;</h2>
            <h2 class="secondary icon" ng-class="forecast.currently.icon | icons"></h2>
            <p>{{forecast.currently.summary}}</p>
            <p>High: {{forecast.daily.data[0].temperatureMax | number:0}}&deg; Low: {{forecast.daily.data[0].temperatureMin | number:0}}&deg; Feels Like: {{forecast.currently.apparentTemperature | number:0}}&deg;</p>
            <p>Wind: {{forecast.currently.windSpeed | number:0}} <span class="icon wind-icon ion-ios7-arrow-thin-up" ng-style="{transform: 'rotate(' + forecast.currently.windBearing + 'deg)'}"></span></p>
          </div>
        </div>
        <div class="scroll-page" ng-style="{width: getWidth(), height: getHeight()}">
          <div class="bar bar-dark">
            <h1 class="title">Daily Forecast</h1>
          </div>
          <div class="has-header">
            <p class="padding">{{forecast.daily.summary}}</p>
            <div class="row" ng-repeat="day in forecast.daily.data | limitTo:settings.days">
              <div class="col col-50">{{day.time + '000' | date:'EEEE'}}</div>
              <div class="col"><span class="icon" ng-class="day.icon | icons"></span><sup>{{day.precipProbability | chance}}</sup></div>
              <div class="col">{{day.temperatureMax | number:0}}&deg;</div>
              <div class="col">{{day.temperatureMin | number:0}}&deg;</div>
            </div>
          </div>
        </div>
        <div class="scroll-page" ng-style="{width: getWidth(), height: getHeight()}">
          <div class="bar bar-dark">
            <h1 class="title">Weather Stats</h1>
          </div>
          <div class="list has-header">
            <div class="item">Sunrise: {{forecast.daily.data[0].sunriseTime | timezone:forecast.timezone}}</div>
            <div class="item">Sunset: {{forecast.daily.data[0].sunsetTime | timezone:forecast.timezone}}</div>
            <div class="item">Visibility: {{forecast.currently.visibility}}</div>
            <div class="item">Humidity: {{forecast.currently.humidity * 100}}%</div>
          </div>
        </div>
      </div>
    </ion-scroll>
  </ion-content>
</ion-view>



```

weather.js

```

angular.module('App')
.controller('WeatherController', function ($scope, $http, $stateParams, $ionicActionSheet, Locations, Settings, $ionicLoading) {
  $scope.params = $stateParams;
  $scope.settings = Settings;
$scope.getWeather = function(){
  $http.get('/api/forecast/' + $stateParams.lat + ',' + $stateParams.lng, {params: {units: Settings.units}}).success(function (forecast) {
    $scope.forecast = forecast;
    $ionicLoading.hide();
  })
  .error(function(err){
    $ionicLoading.show({
      template: 'Could not load weather. Please try again later.',
      duration: 3000
    });
  })
      .finally(function() {
        $scope.$broadcast('scroll.refreshComplete');
      });
  }; 

  $ionicLoading.show();
 
  $scope.getWeather();

  var barHeight = document.getElementsByTagName('ion-header-bar')[0].clientHeight;
  $scope.getWidth = function () {
    return window.innerWidth + 'px';
  };
  $scope.getTotalHeight = function () {
    return parseInt(parseInt($scope.getHeight()) * 3) + 'px';
  };
  $scope.getHeight = function () {
    return parseInt(window.innerHeight - barHeight) + 'px';
  };

  $scope.showOptions = function () {
    var sheet = $ionicActionSheet.show({
      buttons: [
        {text: 'Toggle Favorite'},
        {text: 'Set as Primary'},
        {text: 'Sunrise Sunset Chart'}
      ],
      cancelText: 'Cancel',
      buttonClicked: function (index) {
        if (index === 0) {
          Locations.toggle($stateParams);
        }
        if (index === 1) {
          Locations.primary($stateParams);
        }
        if (index === 2) {
          $scope.showModal();
        }
        return true;
      }
    });
  };
  $scope.showModal = function () {
    if ($scope.modal) {
      $scope.modal.show();
    } else {
      $ionicModal.fromTemplateUrl('views/weather/modal-chart.html', {
        scope: $scope
      }).then(function (modal) {
        $scope.modal = modal;
        var days = [];
        var day = Date.now();
        for (var i = 0; i < 365; i++) {
          day += 1000 * 60 * 60 * 24;
          days.push(SunCalc.getTimes(day, $scope.params.lat, $scope.params.lng));
        }
        $scope.chart = days;
        $scope.modal.show();
      });
    }
  };
  $scope.hideModal = function () {
    $scope.modal.hide();
  };
  $scope.$on('$destroy', function() {
    $scope.modal.remove();
  });
});



```
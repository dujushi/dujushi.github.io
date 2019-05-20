---
title: 'Travel Plan: React Basic'
---
'Travel Plan' is a simple web app I implemented by following the tutorial on React official website. It uses basic React functions and Google Map Javascript APIs. 

The app has a large input field for users to type in their next travel stops with autocomplete support from Google AutocompleteService API. And it has a list of stops with distances and durations between them. The distance and duration data come from Google DistanceMatrixService API.

## PlaceForm
{% highlight js %}
var PlaceForm = React.createClass({
  defaultState: {keywords: '', predictions: [], error: ''},
  getInitialState: function() {
    return this.defaultState;
  },
  render: function() {
    var predictionNodes = this.state.predictions.map(function(prediction) {
      return (
        <li key={prediction.place_id}
            onClick={this.handlePlaceSelect.bind(this, prediction.place_id, prediction.description)}
            >
          {prediction.description}
        </li>
      );
    }.bind(this));
    return (
      <div>
        <input value={this.state.keywords} onChange={this.handleKeywordsChange} placeholder='Enter your next stop' />
        {this.state.error != "" ? <div className='Error'>{this.state.error}</div> : null}
        {this.state.predictions.length == 0 ? null : <ul>{predictionNodes}</ul>}
      </div>
    );
  },
  handleKeywordsChange: function(e) {
    var keywords = e.target.value;
    this.setState({keywords: keywords});

    // no query if keywords has fewer than 5 characters
    if (keywords.length < 5) {
      this.setState({predictions: []});
    } else {
      var service = new google.maps.places.AutocompleteService();
      service.getQueryPredictions({input: keywords}, function(predictions, status) {
        // result may come back in different order.
        // only show results for current keywords.
        if (keywords != this.state.keywords) return;

        switch (status) {
          case google.maps.places.PlacesServiceStatus.OK:
            this.setState({predictions: predictions});
            break;
          case google.maps.places.PlacesServiceStatus.ZERO_RESULTS:
            this.setState({predictions: []});
            break;
          default:
            this.setState({error: status, predictions: []});
            break;
        }
      }.bind(this));
    }
  },
  handlePlaceSelect: function(place_id, description, e) {
    this.setState(this.defaultState);
    this.props.onPlaceSelect({place_id: place_id, description: description, duration: '', distance: ''});
  }
});
{% endhighlight %}
 PlaceForm component takes one onPlaceSelect property and has three state attributes (keywords, predictions, error). 'keywords' stores current user input. 'predictions' stores all places returned from Google AutocompleteService. 'error' keeps error from the Google API. We use *bind(this)* for both callback functions so that we can set state within the callback. And we use *bind(this, prediction.place_id, prediction.description)* to pass place info to handlePlaceSelect. The callback function is defined as *function(place_id, description, e)*. Notice that place info parameters should be added before the default event parameter.
## PlaceBox
{% highlight js %}
var PlaceBox = React.createClass({
  getInitialState: function() {
    return {places: []};
  },
  render: function() {
    return (
      <div className='PlaceBox'>
        <h1>Travel Plan</h1>
        <PlaceList places={this.state.places}/>
        <PlaceForm onPlaceSelect={this.handlePlaceSelect}/>
      </div>
    );
  },
  handlePlaceSelect: function(place) {
    var places = this.state.places;
    // check if place_id already exists
    var found = places.find(function(element, index, array) {
      return element.place_id == place.place_id;
    });
    if (found === undefined) {
      // calculate distance and duration from last stop
      if (places.length > 0) {
        var lastPlace = places[places.length - 1];
        lastPlace.distance = 'Calculating';
        lastPlace.duration = 'Calculating';
        places[places.length - 1] = lastPlace;

        this.calculateDistanceMatrix(lastPlace, place);
      }
      places.push(place);
      this.setState({places: places});
    }
  },
  calculateDistanceMatrix: function(originPlace, destinationPlace) {
    var service = new google.maps.DistanceMatrixService;
    service.getDistanceMatrix({
      origins: [originPlace.description],
      destinations: [destinationPlace.description],
      travelMode: google.maps.TravelMode.DRIVING,
      unitSystem: google.maps.UnitSystem.METRIC,
      avoidHighways: false,
      avoidTolls: false
    }, function(response, status) {
      if (status !== google.maps.DistanceMatrixStatus.OK) {
        alert('Error! DistanceMatrixStatus: ' + status);
      } else {
        var results = response.rows[0].elements;
        var status = results[0].status;
        if (status == google.maps.DistanceMatrixElementStatus.OK) {
          originPlace.distance = results[0].distance.text;
          originPlace.duration = results[0].duration.text;
        } else {
          originPlace.distance = status;
          originPlace.duration = status;
        }
        var places = this.state.places;
        // find the right place to update distance and duration
        var found = places.find(function(element, index, array) {
          return element.place_id == originPlace.place_id;
        });
        if (found != undefined) {
          places[found] = originPlace;
          this.setState({places: places});
        }
      }
    }.bind(this));
  }
});
{% endhighlight %}
 handlePlaceSelect receives place info from PlaceForm. It uses *find* function to check if the place has been added. If not, *find* will return undefined and we add the place info to our place list. If there is a previous stop, we pass both places to calculateDistanceMatrix to retrieve distance and duration info.
## PlaceList
{% highlight js %}
var PlaceList = React.createClass({
  render: function() {
    var placeNodes = this.props.places.map(function(place) {
      return (
        <Place key={place.place_id} {...place} />
      );
    });
    return (
      <div className='PlaceList'>
        {placeNodes}
      </div>
    );
  }
});
{% endhighlight %}
 Instead of adding each property separately for Place node, we use spread attribute feature *{...place}*.
## Place
{% highlight js %}
var Place = React.createClass({
  render: function() {
    var distanceMatrixNode = '';
    var isCalculating = this.props.distance == 'Calculating';
    if (this.props.distance != "") {
      distanceMatrixNode = (
        <div className='DistanceMatrix'>
          <span>{!isCalculating ? this.props.distance : <i className='fa fa-spinner fa-spin'></i>}</span>
          <i className='fa fa-long-arrow-down ToNextStop'></i>
          <span>{!isCalculating ? this.props.duration : <i className='fa fa-spinner fa-spin'></i>}</span>
        </div>
      );
    }
    return (
      <div className='Place'>
        <h2 className='Stop'>
          <i className='fa fa-car'></i>
          <span>{this.props.description}</span>
        </h2>
        {distanceMatrixNode}
      </div>
    );
  }
});
{% endhighlight %}
 We use some fancy icons from FontAwesome.
## Render
{% highlight js %}
ReactDOM.render(
  <PlaceBox />,
  document.getElementById('content')
);
{% endhighlight %}
 We add 'Powered by Google' at the end of the page to acknowledge Google's contribution to this app:P
## Github
[https://github.com/dujushi/snippets/tree/master/react-basic-google-distance-matrix](https://github.com/dujushi/snippets/tree/master/react-basic-google-distance-matrix){:target="_blank"}

## References
1. [React Tutorial](https://facebook.github.io/react/docs/tutorial.html){:target="_blank"}
2. [Retrieving Autocomplete Predictions](https://developers.google.com/maps/documentation/javascript/examples/places-queryprediction){:target="_blank"}
3. [Distance Matrix service](https://developers.google.com/maps/documentation/javascript/examples/distance-matrix){:target="_blank"}
4. [Google Maps JavaScript API V3 Reference](https://developers.google.com/maps/documentation/javascript/3.exp/reference#DistanceMatrixElementStatus){:target="_blank"}
5. [Pass argument to reactjs click handler](http://stackoverflow.com/questions/29188326/pass-argument-to-reactjs-click-handler){:target="_blank"}

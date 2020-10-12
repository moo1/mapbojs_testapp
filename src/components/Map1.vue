<template>
    <div id='map' style='width: 600px; height: 480px;'></div>
</template>


<script>
  import mapboxgl from 'mapbox-gl';
  import axios from 'axios';

  var MapMixin =  {
    name: 'Map1',
    props: {
      msg: String,
    },
    $_map: null,  // does this help?
    $_pois:[], // does this help?
    mounted(){
      // Fetch data from data server.
      this.$_pois = [];
      fetch_data(this);
      // init Mapbox
      mapboxgl.accessToken = 'pk.eyJ1IjoibW9vMSIsImEiOiJja2Z6MG5tZzgwaHJjMzRudHdqY3N5aG05In0.L_YIvkoB67ATZVDHOZXDSg';
      this.$_map = new mapboxgl.Map({
          container: 'map',
          style: 'mapbox://styles/mapbox/streets-v11', // stylesheet location
          center: [-74.5, 40], // starting position [lng, lat]
          zoom: 10 // starting zoom
      });
      var nav = new mapboxgl.NavigationControl();
      this.$_map.addControl(nav, 'top-left');

    }
  }

  function fetch_data(obj) {
    const data_url = 'https://gist.githubusercontent.com/lbud/35e4847d13e5524d08d3e547318cf689/raw/d44940c13e70b2bb683345c511f85d249ee5ccfc/starbucks.csv';
    axios.get(data_url, {
      params: {},
    }).then(({ data }) => {
      // split into lines
      let lines = 0;
      for (var l of data.split("\n")) {
        // skip the first line
        if (lines>0) {
          let sb = {
            phone_number: '',
            street: '',
            city: '',
            lat: 0.0,
            lng: 0.0
          };
          // split by comma
          let items = parse_csv(l);
          //console.log(items);
          if (items.length==19) {
            sb.phone_number = items[5];
            sb.street = items[7];
            sb.city = items[11];
            sb.lat = parseFloat(items[14]);
            sb.lng = parseFloat(items[15]);
            obj.$_pois.push(sb);
            console.log(sb);
            // adding a marker
            var marker = new mapboxgl.Marker({color:'#00704a'});
            marker.setLngLat([sb.lng, sb.lat]);
            marker.setPopup(new mapboxgl.Popup().
              setHTML(`<div class="poipopup">Address: ${sb.street}</div>
                       <div class="poipopup">Phone: ${sb.phone_number}</div`));
            marker.addTo(obj.$_map);
          }
        }
        lines++;
      }
      // move the center
      let c = center(obj.$_pois);
      obj.$_map.setCenter([c[1], c[0]]);
    });

  }

  // average of Lat/Lng
  function center(pois) {
    let n = pois.length;
    let lat = 0.0;
    let lng = 0.0;
    for (let p of pois) {
      lat += p.lat;
      lng += p.lng;
    }
    return [lat/n, lng/n];
  }

  //
  function parse_csv(str) {
    let cs = str.split(',');
    let tmps = "";
    let result = [];
    for (var itm of cs) {
      let s = itm.trim();
      if (tmps.length>0) {
        tmps += s;
        if (s.charAt(s.length-1)==='"') {
          // closing quote
          result.push(tmps.substr(0, s.length-2));
          tmps = '';
        }
      } else if (s.charAt(0)==='"') {
        // starting quote
        tmps = s.substr(1, s.length-1);
      } else {
        result.push(s);
      }
    }
    return result;
  }
  // test...
  //console.log(parse_csv('US,15298,Grand Central,Starbucks,7800-67600,212-370-4161,CO,"7800 Grand Central Station, Track 35, Grand Central Station",7800 Grand Central Station,Track 35,Grand Central Station,New York,NY,10017,40.753669738769531,-73.976791381835938,Eastern Standard Time,-300,GMT-05:00 America/New_York'));
  //console.log(parse_csv('US,6590,88th & Broadway,Starbucks,14498-121264,212-595-3404,CO,2394 Broadway,2394 Broadway,,,New York,NY,100241703,40.789287567138672,-73.975326538085938,Eastern Standard Time,-300,GMT-05:00 America/New_York'));

  export default MapMixin;
</script>

<style >
@import 'https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css';
</style>

<style >
.poipopup {
  font-size: 0.9em;
  color:#777;
}
</style>

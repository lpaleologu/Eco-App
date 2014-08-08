Eco-App
=======
<!DOCTYPE html>
<html>
<head>
    <title>Walking</title>
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no">
    <meta charset="utf-8">
    <style>
        html, body, #map-canvas {
            height: 100%;
            margin: 0px;
            padding: 0px
        }
    </style>
    <script src='https://maps.googleapis.com/maps/api/js?v=3.exp'></script>

    <script src='https://cdn.firebase.com/js/client/1.0.15/firebase.js'></script>
   <script src="http://code.jquery.com/jquery-1.9.1.min.js"></script>
    <script src="http://code.jquery.com/mobile/1.3.1/jquery.mobile-1.3.1.min.js"></script>

</head>


<body>
<div datarole="page" id="map">
<div id="buttons"></div>
<div  id="map-canvas" style="width:600px; height:600px;"></div>


    <script>
        // Note: This example requires that you consent to location sharing when
        // prompted by your browser. If you see a blank space instead of the map, this
        // is probably because you have denied permission for location sharing.



        var map;
        $(document).ready(function(){

            $('<input />', { type: 'button', value: 'start', onclick: 'StartClicked()'}).appendTo($('#buttons'));
            $('<input/>', {type: 'button', value: 'end', onclick: 'EndClicked()'}).appendTo($('#buttons'));

        });

        function initialize() {
            var mapOptions = {
                zoom: 18
            };
            map = new google.maps.Map(document.getElementById('map-canvas'),
                    mapOptions);

            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(function (position) {
                    var pos = new google.maps.LatLng(position.coords.latitude,
                            position.coords.longitude)});
            }
                map.setCenter(pos);



        }

        function handleNoGeolocation(errorFlag) {
            if (errorFlag) {
                var content = 'Error: The Geolocation service failed.';
            } else {
                var content = 'Error: Your browser doesn\'t support geolocation.';
            }

            var options = {
                map: map,
                position: new google.maps.LatLng(60, 105),
                content: content
            };

            var infowindow = new google.maps.InfoWindow(options);
            map.setCenter(options.position);
        }


        function distance(pos1,pos2){

            var R = 6371; // km
            var φ1 = pos1[0]*(Math.PI/180);
            var φ2 = pos2[0]*(Math.PI/180);
            var Δφ = (pos2[0]-pos1[0])*(Math.PI/180);
            var Δλ = (pos2[1]-pos1[1])*(Math.PI/180);

            var a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
                    Math.cos(φ1) * Math.cos(φ2) *
                    Math.sin(Δλ/2) * Math.sin(Δλ/2);
            var c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

            var d = R * c;
            return d;
        }




        var i;
        var distanceList=[];
        var posList=[];
        var Total=0;
        var myDataRef = new Firebase('https://ecoappgps.firebaseio.com/');
        var timing;
        //var pos;
       // var InfoWindow;
        var infoList=[];


        //console.log(pos);







        //console.log('buttons appear');
        function StartClicked() {
          i=0;
          myDataRef.set(null);
            if (i === 0) {

               timing = setInterval(function () {


                    if (navigator.geolocation) {
                        navigator.geolocation.getCurrentPosition(function (position) {
                            var pos = new google.maps.LatLng(position.coords.latitude,
                                    position.coords.longitude);
                            //console.log(pos);

                            if (infoList.length>0) {
                                infoList[infoList.length-1].close();
                            }
                                var InfoWindow = new google.maps.InfoWindow({
                                    map: map,
                                    position: pos,
                                    content: 'Location found using HTML5.'
                                });

                            infoList.push(InfoWindow);




                            map.setCenter(pos);


                            myDataRef.push({positionLat: pos.lat(), positionLng: pos.lng()});
                            myDataRef.on('child_added', function (snapshot) {
                                //We'll fill this in later.
                                var message = snapshot.val();

                                //console.log(message);


                                posList.push([message.positionLat, message.positionLng]);
                                console.log(posList);

                               if (posList.length>1) {
                                   var flightPath = new google.maps.Polyline({
                                       path: [posList[posList.indexOf(pos)], posList[posList.indexOf(pos)-1]],
                                       geodesic: true,
                                       strokeColor: '#FF0000',
                                       strokeOpacity: 1.0,
                                       strokeWeight: 6
                                   });
                                   flightPath.setMap(map);
                               }


                            });





                            /*InfoWindow.position_changed=function(){
                                this.close();
                            }*/

                        }, function () {
                            handleNoGeolocation(true);
                        });
                    } else {
                        // Browser doesn't support Geolocation
                        handleNoGeolocation(false);
                    }




                }, 3000);




                //google.maps.event.addDomListener(window, 'load', initialize);


            }

        }

        function EndClicked() {
            i = 1;
            clearInterval(timing);
            timing=0;
            //console.log('end button clicked');

            for (var k = 0; k < posList.length; k++) {
                if (k > 0) {
                    var posDistance = distance(posList[k], posList[k - 1]);
                    //var posSpeed=posDistance/3;
                    /* if (posSpeed>=0.0044704){

                     alert('You are riding in a vehicle!');
                     }*/

                    distanceList.push(posDistance);

                }
                //console.log(distanceList);
            }
            for (var j=0; j<distanceList.length-1; j++){
                Total=Total+distanceList[j];


            }
            posList=[];
            distanceList=[];

            console.log(Total);
            alert(Total + ' km');

            Total=0;
            console.log(Total);
            myDataRef.set(null);
        }






           /* if (i === 0) {
                console.log('i=0');
                setTimeout(function () {
                }, 5000);
                google.maps.event.addDomListener(window, 'load', initialize);

            }*/


        google.maps.event.addDomListener(window, 'load', initialize);



    </script>








</div>
</body>


</html>

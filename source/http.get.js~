var http = require('http');

var option = {
  host: 'nb.gl',
}

http.get(option, function(res){
  var data = '';
  console.log(res.headers);
  res.on('data', function(trunk){
    data += trunk;
  });
  res.on('end', function(trunk){
    if (trunk) {
      data += trunk;
    }
    console.log(data);
  });
});

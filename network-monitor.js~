var fs = require('fs');
var moment = require('moment');

/** This module requires the "sysstat" package to be installed. It can be installed via the "apt-get install sysstat" command. */
function NetworkTrafficMonitor() {
    var _this = this;
    var chronicJob = null;
    var listeners = [];

    /** samplingRate should be in msec */
    this.startListening = function(samplingRate) {
    	_this.stopListening();
    	var savedDeviceData = {};
        var chronicListeningJob = function() {
        	var dataToReport = {};
            fs.readFile('/proc/net/dev', function(err, data) {
            	if (err) {
            		throw err;
            	}

            	var dataTime = moment();
            	var lines = data.toString().split("\n");
            	lines = lines.slice(2);
            	lines.forEach(function(line) {
            		var regex = /\s*(.*)\:\s(\d*)(\s*\d*){8}\s(\d+)/;
            		var matches = regex.exec(line);
            		if (matches) {
            			var name = matches[1];
            			var readBytes = parseInt(matches[2]);
            			var writenBytes = parseInt(matches[4]);

            			if (savedDeviceData[name]) {
            				var diffSeconds = dataTime.diff(savedDeviceData[name].time, 'seconds');
            				var diffReadBytes = readBytes - savedDeviceData[name].readBytes;
            				var diffWritenBytes = writenBytes - savedDeviceData[name].writenBytes;

            				dataToReport[name] = new TrafficData(name, diffReadBytes / diffSeconds, diffWritenBytes / diffSeconds);
            			}
            			savedDeviceData[name] = new AuxTrafficData(dataTime, readBytes, writenBytes);
            		}
            	});

            	listeners.forEach(function(callback) {
            	    callback(dataToReport);
            	});

            	chronicJob = setTimeout(chronicListeningJob, samplingRate);
            });
        };

        chronicListeningJob();
        
        return _this;
    };

    this.stopListening = function() {
        if (chronicJob) {
            clearTimeout(chronicJob);
        }
        return _this;
    };

    /** callback should be on the form of function (TrafficData) {...} */
    this.addListener = function(callback) {
        listeners.push(callback);
        return _this;
    };

    this.removeListener = function(callback) {
        listeners.splice(listeners.indexOf(callback), 1);
        return _this;
    };
}

function AuxTrafficData(time, readBytes, writenBytes) {
	this.time = time;
	this.readBytes = readBytes;
	this.writenBytes = writenBytes;
}

function TrafficData(device, readKBps, writeKBps) {
    this.device = device;
    this.readKBps = readKBps;
    this.writeKBps = writeKBps;
}

module.exports = NetworkTrafficMonitor;

# TW-Coordgrabber
Voeg onderstaande toe aan snelschakelijst


javascript:

$("#ds_body").append('<div id="DS_border" style="display: block; z-index: 10000; position: absolute; width: 0px; height: 0px; white-space: nowrap; left: 554px; top: 126px; opacity: 0.5; background-color:yellow;"/>');
var DS_Map = TWMap;
var x = '',
y = '',
xi = '',
xy = '',
busy = false,
sndbusy = false,
thdbusy = true, // set to true for tribalwars 8.29.1 -> shift doesnt need to be pressed to select/deselect coord with click
arr = [],
el = $("#DS_border");
function spawnSectorReplacer(data, sector) {
DS_Map.mapHandler.DSspawnSector(data, sector);
var beginX = sector.x - data.x;
var endX = beginX + DS_Map.mapSubSectorSize;
var beginY = sector.y - data.y;
var endY = beginY + DS_Map.mapSubSectorSize;
for (var x in data.tiles) {
var x = parseInt(x, 10);
if (x < beginX || x >= endX) {
continue;
}
for (var y in data.tiles[x]) {
var y = parseInt(y, 10);

if (y < beginY || y >= endY) {
continue;
}
var xCoord = data.x + x;
var yCoord = data.y + y;
var v = DS_Map.villages[(xCoord) * 1000 + yCoord];
if (v) {
var pl = (v.owner > 0 && TWMap.players[v.owner]) ? TWMap.players[v.owner] : 0;
var overlay = document.createElement('div');
overlay.style.position = 'absolute';
overlay.style.zIndex = '50';
overlay.style.width = (DS_Map.map.scale[0] - 1).toString() + 'px';
overlay.style.height = (DS_Map.map.scale[1] - 1).toString() + 'px';
overlay.style.opacity = 0.3;
overlay.style.background = $("#DS_coordlist").text().match(xCoord + "\\|" + yCoord) ? "blue" : "none";
overlay.className = "DS_overlay";
overlay.id = ['DSoverlay', xCoord + "|" + yCoord, v.owner, pl.ally, pl != 0 ? pl.name : ''].join("_");
sector.appendElement(overlay, x - beginX, y - beginY);
}
}
}
}
DS_Map.mapHandler.DSspawnSector = DS_Map.mapHandler.spawnSector;
TWMap.mapHandler.spawnSector = spawnSectorReplacer;
DS_Map.map._DShandleClick = DS_Map.map._handleClick;	
TWMap.map._handleClick = function (e) {
if (thdbusy) {
var pos = this.coordByEvent(e),
coord = pos.join("|"),
coords = $("#DS_coordlist").val().match(/\d{1,3}\|\d{1,3}/g) || [],
v = TWMap.villages[(pos[0]) * 1000 + pos[1]];

if (v) {
var ii = coords.indexOf(coord);
if (ii >= 0) {
coords.splice(ii, 1);
$('[id*="' + coord + '"]').css("background", "none")
} else {
coords.push(coord);
$('[id*="' + coord + '"]').css("background", "blue")
}
Refresh(coords);
return false;
}
} else {
DS_Map.map._DShandleClick(e);
return false;
}
};
TWMap.reload();

function Refresh(coords) {
$("#DS_coordlist").text($.map((coords || $("#DS_coordlist").text().match(/\d{1,3}\|\d{1,3}/g)), function (e) {
return $("#DS_bbcodes").is(":checked") ? 'Dorp niet beschikbaar' : e;
}).join('\n'));

var new_count = ($('#DS_coordlist').val() != '') ? $('#DS_coordlist').val().match(/\d{1,3}\|\d{1,3}/g).length : '0';
$('.select_count').text(new_count);
}

function exit() {
if (busy) {
TWMap = func;
$('[id*="DSoverlay_"]').each(function () {
var offset = $(this).offset(),
arr = this.id.split("_"),
owner = arr[2],
n = $(".DS_opts:checked").attr("data");
if (
(owner == 0 && n == 3) || 
(owner == game_data.player.id && n == 4) || 
(n == 5 && owner != game_data.player.id && owner != 0) || 
(n == 6 && arr[3] != 0 && arr[3] != game_data.player.ally) || 
(n == 7 && owner != 0 && document.getElementById("DS_byname").value.match(RegExp(arr[4], "i"))) || n == 1 || n == 2) {
if ((offset.left + this.offsetWidth) > xi && offset.left < (xi + $("#DS_border").width()) && offset.top > yi && offset.top < (yi + $("#DS_border").height())) {
var coord = this.id.split("_")[1],
coords = $("#DS_coordlist")[0].textContent.match(/\d{1,3}\|\d{1,3}/g) || [];
var ii = coords.indexOf(coord);
if (n == 2 && ii != -1) {
coords.splice(ii, 1);
$(this).css("background", 'none');
} else if (n != 2 && ii == -1) {
$(this).css("background", 'blue');
coords.push(coord);
}
Refresh(coords);
}
}
});
$("#DS_border").width(0).height(0);
!$('#show_popup').is(':checked') && $('#show_popup').click();
$(".autoHideBox").css("opacity", 0);
$('#map_mover').show();
x = '';
y = '';
busy = false;
}
}
func = TWMap;
$("#ds_body").keydown(function (e) {
if (e.which == 16) {
busy = true;
thdbusy = true;
$('#map_mover').hide();
}
}).keyup(function (e) {
sndbusy = false;
e.which == 16 && (thdbusy = true); // set to true for tribalwars 8.29.1 -> shift doesnt need to be pressed to select/deselect coord with click
exit();
return false;
}).mousedown(function (e) {
if (busy) {
sndbusy = true;
$("#DS_border").offset({
top: e.pageY,
left: e.pageX
});
x = e.pageX;
y = e.pageY;
$('#map_mover').hide();
$('#show_popup').is(':checked') && $('#show_popup').click();
$(".autoHideBox").css("opacity", 0);
return false;
}
}).mouseup(function () {
sndbusy = false;
exit();
return false;
});
document.onmousemove = function (a) {
if ("" != x && "" != y && busy && sndbusy) {
tx = -x + a.pageX;
ty = -y + a.pageY;
el.width(0 > tx ? -tx : tx).height(0 > ty ? -ty : ty).offset({
top: 0 > ty ? a.pageY : y,
left: 0 > tx ? a.pageX : x
});
xi = 0 > tx ? a.pageX : x;
yi = 0 > ty ? a.pageY : y;
var b = xi + el.width(),
c = yi + el.height(),
d = document.getElementsByClassName("DS_overlay"),
n = $(".DS_opts:checked").attr("data");
if (!$("#DS_mousemove").is(":checked")) { for (i = 0; i < d.length; i++) {
var e = $(d).offset(),
f = document.getElementById("DS_coordlist").textContent.indexOf(d.id.split("_")[1]),
arr = d.id.split("_"),
owner = arr[2];
if (
(owner == 0 && n == 3) || 
(owner == game_data.player.id && n == 4) || 
(n == 5 && owner != game_data.player.id && owner != 0) || 
(n == 6 && arr[3] != 0 && arr[3] != game_data.player.ally) || 
(n == 7 && owner != 0 && document.getElementById("DS_byname").value.match(RegExp(arr[4], "i"))) || n == 1 || n == 2) {
(e.left + d.offsetWidth) > xi && e.left < b && e.top > yi && e.top < c ? n == 2 ? 
d.style.background = "none" : -1 == f && (d.style.background = "blue") : -1 == f ? d.style.background = "none" : -1 != f && (d.style.background = "blue");
}
}
};
return false;
};
};

function get_browserinfo(){
var ua=navigator.userAgent,tem,M=ua.match(/(opera|chrome|safari|firefox|msie|trident(?=\/))\/?\s*(\d+)/i) || []; 
if(/trident/i.test(M[1])){
tem=/\brv[ :]+(\d+)/g.exec(ua) || []; 
return 'IE '+(tem[1]||'');
} 
if(M[1]==='Chrome'){
tem=ua.match(/\bOPR\/(\d+)/)
if(tem!=null) {return 'Opera '+tem[1];}
} 
M=M[2]? [M[1], M[2]]: [navigator.appName, navigator.appVersion, '-?'];
if((tem=ua.match(/version\/(\d+)/i))!=null) {M.splice(1,1,tem[1]);}
return [M[0], M[1]];
}

var bi = get_browserinfo();
var str = '<br/><table class="vis" style="border-spacing:0px;border-collapse:collapse;" width="100%"><tbody>';
str += '<tr><th colspan="100%">Coord grabber </th></tr>';
str += '<tr><td><b><span class="select_count">0</select></b> dorpen geselecteerd<br><textarea style="height: 160px; width: 180px;" id="DS_coordlist" onfocus="this.select();"/></td><td>';
str += '<input type="checkbox" id="DS_bbcodes" data="0"> Coordinaten in BB codes</input><br/>';
if (bi[0] == 'Opera' && parseInt(bi[1]) <= 12) {
str += '<input type="checkbox" id="DS_mousemove" checked="checked" data="0"> Aanvinken voor tragere computers / oude versie van opera</input><br/>';
} else { 
str += '<input type="checkbox" id="DS_mousemove" data="0"> Aanvinken voor tragere computers / oude versie van opera</input><br/>';
}
str += '<input name="selectors" type="radio" class="DS_opts" data="1"> Selecteer alle dorpen</input><br/>';
str += '<input name="selectors" type="radio" class="DS_opts" data="2"> Coordinaten verwijderen</input><br/>';
str += '<input name="selectors" type="radio" class="DS_opts" data="3"> Selecteer alleen barbarendorpen</input><br/>';
str += '<input name="selectors" type="radio" class="DS_opts" data="4"> Selecteer dorpen van jezelf</input><br/>';
str += '<input name="selectors" type="radio" class="DS_opts" data="5"> Selecteer dorpen van andere spelers</input><br/>';
str += '<input name="selectors" type="radio" class="DS_opts" data="6"> Selecteer dorpen van niet-stamgenoten</input><br/>';
str += '<input name="selectors" type="radio" class="DS_opts" data="7"><input type="text" id="DS_byname" value=""></input> Selecteer alleen dorpen van deze speler</input><br/>';
str += '<input style="display:none" type="checkbox" id="DS_oneclick" data="8"><br/>';
str += '</td></tr></tbody></table>';
$("#map_config").before(str);
$("#DS_bbcodes").change(function () {
Refresh();
});
$(".DS_opts:first").click();
void(0);

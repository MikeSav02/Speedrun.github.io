<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Speedrun Challenge</title>
<base href="./">
<style>
body{font-family:"Segoe UI","Roboto","Helvetica Neue",Arial,sans-serif;background:#050814;color:#e8ecff;margin:0;overflow-x:hidden}
body::before{content:"";position:fixed;inset:0;background-image:linear-gradient(#ff8c4222 1px,transparent 1px),linear-gradient(90deg,#ff8c4222 1px,transparent 1px);background-size:80px 80px;opacity:.25;animation:gridDrift 60s linear infinite;z-index:-1}
@keyframes gridDrift{0%{transform:translate3d(0,0,0)}100%{transform:translate3d(-80px,-80px,0)}}

#stickyTop{position:sticky;top:0;z-index:9999;background:#050814;box-shadow:0 4px 12px #00000088}

#stickyControls{background:#050814;padding:12px 22px;border-bottom:1px solid #13243d;display:flex;flex-wrap:wrap;gap:12px;align-items:center;position:sticky;top:var(--sticky-controls-top,0px);z-index:9998}
#stickyControls input[type=text],#stickyControls input[type=date]{padding:8px 10px;font-size:14px;border-radius:4px;border:1px solid #29405f;background:#050814;color:#e8ecff}
#stickyControls button{padding:8px 14px;border-radius:4px;background:#07101f;border:1px solid #ff8c42;color:#ff8c42;font-size:14px;cursor:pointer;text-transform:uppercase;letter-spacing:.06em}
#stickyControls button:hover{background:#0c1a2b}

#dailyGoalContainer{margin-top:6px;width:100%}
#dailyGoalContainer>div:first-child{font-size:14px;color:#4fc3f7}
#dailyGoalBar{width:100%;height:18px;background:#0b1628;border-radius:6px;overflow:hidden;border:1px solid #1f2a44;margin-top:4px}
#dailyGoalFill{height:100%;width:0;background:#4caf50;transition:width .3s ease}
#dailyGoalStatus{margin-top:4px;font-size:14px;color:#4fc3f7}


#totalProgressContainer{margin-top:6px;width:100%}
#totalProgressContainer>div:first-child{font-size:14px;color:#ffcc80}
#totalProgressBar{width:100%;height:18px;background:#0b1628;border-radius:6px;overflow:hidden;border:1px solid #1f2a44;margin-top:4px}
#totalProgressFill{height:100%;width:0;background:#4fc3f7;transition:width .3s ease}
#totalProgressStatus{margin-top:4px;font-size:14px;color:#4fc3f7}


main{padding:18px 22px}
/*.summary-line{margin-bottom:18px;color:#4fc3f8;font-size:15px}*/

.entry-card{border-radius:6px;padding:12px 14px;margin-bottom:12px;box-shadow:0 0 8px #0d1324 inset;transition:background .2s ease,transform .1s ease,opacity .2s ease;border:2px solid transparent;position:relative}
.entry-card::before{content:"";position:absolute;inset:4px;border:1px solid rgba(255,140,66,.18);pointer-events:none}
.entry-card.done{opacity:.55;text-decoration:line-through}
.entry-card:active{transform:scale(.98)}
.entry-title{font-size:17px;font-weight:600}
.entry-meta{font-size:14px;margin-top:4px}
.badge{padding:2px 6px;border-radius:4px;font-size:11px;background:#ff8c4215;color:#ffcc80}

/* MOBILE OPTIMIZATION */
@media (max-width:600px){
#stickyControls{padding:8px 12px;gap:6px}
#stickyControls input[type=text],#stickyControls input[type=date]{padding:6px 8px;font-size:12px}
#stickyControls button{padding:6px 10px;font-size:12px}
#dailyGoalContainer{margin-top:4px}
#dailyGoalBar{height:12px}
#dailyGoalStatus,#liveCountdown,#currentPace{font-size:12px}
#stickyTop{box-shadow:0 2px 6px #00000088}
.entry-card{padding:10px 12px;margin-bottom:8px;border-width:1px;border-radius:5px;box-shadow:0 0 4px #0d1324 inset}
.entry-title{font-size:15px;line-height:1.25}
.entry-meta{font-size:12px;margin-top:2px;line-height:1.2}
.badge{font-size:10px;padding:1px 4px;border-radius:3px}
.entry-card:active{transform:scale(.97)}
}
</style>
</head>
<body>

<div id="stickyTop">
<div id="stickyControls">
<input id="searchInput" type="text" placeholder="Search…">
<input id="startDateInput" type="date">
<input id="endDateInput" type="date">
<button id="calcBtn">Set Goal</button>
<button id="resetDayBtn">Reset Day Stats</button>
<button id="clearDoneBtn">Clear All</button>

<div id="totalProgressContainer">
<div>Total Runtime Progress</div>
<div id="totalProgressBar"><div id="totalProgressFill"></div></div>
<div id="liveCountdown" style="margin-top:6px;color:#4fc3f7;font-size:14px;font-family:Consolas,monospace;"></div>
<div id="totalProgressStatus"></div>
</div>

<div id="dailyGoalContainer">
<div style="font-size:14px;color:#ffcc80;">Daily Goal Progress</div>
<div id="dailyGoalBar"><div id="dailyGoalFill"></div></div>
<div id="dailyGoalStatus"></div>

<div id="currentPace" style="margin-top:6px;color:#4fc3f7;font-size:14px;"></div>

</div>
</div>
</div>

<main>
<div class="summary-line" id="summaryLine"></div>
<div id="cardContainer"></div>
</main>
<script>
const KEY="sw_done_map_v2",DAY_KEY="sw_day_reset_timestamp",DAY_HISTORY_KEY="sw_day_history",PREV_MIN_KEY="sw_prev_minutes";
let doneMap=JSON.parse(localStorage.getItem(KEY)||"{}"),
    dayReset=Number(localStorage.getItem(DAY_KEY)||Date.now()),
    dayHistory=JSON.parse(localStorage.getItem(DAY_HISTORY_KEY)||"[]"),
    previousMinutes=Number(localStorage.getItem(PREV_MIN_KEY)||0),
    lastPerDayTarget=0;

// ── Cached DOM references ─────────────────────────────────────────────────────
const elSummaryLine    = document.getElementById("summaryLine");
const elCardContainer  = document.getElementById("cardContainer");
const elTotalProgressFill = document.getElementById("totalProgressFill");
const elTotalProgressStatus = document.getElementById("totalProgressStatus");
const elDailyGoalFill  = document.getElementById("dailyGoalFill");
const elDailyGoalStatus= document.getElementById("dailyGoalStatus");
const elLiveCountdown  = document.getElementById("liveCountdown");
const elCurrentPace    = document.getElementById("currentPace");
const elSearchInput    = document.getElementById("searchInput");
const elStartDate      = document.getElementById("startDateInput");
const elEndDate        = document.getElementById("endDateInput");
const elStickyTop      = document.getElementById("stickyTop");
const elStickyControls = document.getElementById("stickyControls");

// ── Dynamic sticky offset (fix: no hardcoded 110px) ──────────────────────────
function updateStickyOffset(){
  document.documentElement.style.setProperty(
    "--sticky-controls-top", elStickyTop.offsetHeight+"px"
  );
}
const resizeObserver=new ResizeObserver(updateStickyOffset);
resizeObserver.observe(elStickyTop);
updateStickyOffset();

const rows=[
['Between c. 68 58 BBY','Tales of the Jedi - Justice','1','1',20],
['','Tales of the Underworld - The Good Life','1','1',20],
['Between c. 50 BBY and 48 BBY','Tales of the Jedi - Choices','1','2',20],
['','Tales of the Underworld - A Good Turn','1','2',20],
['36 35 BBY','Tales of the Jedi - Life and Death','1','3',20],
['','Tales of the Underworld - One Good Deed','1','3',20],
['32 BBY','Episode I The Phantom Menace','','',136],
['32 BBY','Tales of the Jedi - The Sith Lord','1','4',20],
['22 BBY','Episode II Attack of the Clones','','',142],
['22 BBY','The Clone Wars - Cat and Mouse','2','16',24],
['22 BBY','The Clone Wars - The Hidden Enemy','1','16',24],
['22 BBY','The Clone Wars film','','',90],
['22 BBY','The Clone Wars - Clone Cadets','3','1',24],
['22 BBY','Tales of the Jedi - Practice Makes Perfect','1','5',20],
['22 BBY','The Clone Wars - Supply Lines','3','3',24],
['22 BBY','The Clone Wars - Ambush','1','1',24],
['22 BBY','The Clone Wars - Rising Malevolence','1','2',24],
['22 BBY','The Clone Wars - Shadow of Malevolence','1','3',24],
['22 BBY','The Clone Wars - Destroy Malevolence','1','4',24],
['22 BBY','The Clone Wars - Rookies','1','5',24],
['22 BBY','The Clone Wars - Downfall of a Droid','1','6',24],
['22 BBY','The Clone Wars - Duel of the Droids','1','7',24],
['22 BBY','The Clone Wars - Bombad Jedi','1','8',24],
['21 BBY','The Clone Wars - Cloak of Darkness','1','9',24],
['21 BBY','The Clone Wars - Lair of Grievous','1','10',24],
['21 BBY','The Clone Wars - Dooku Captured','1','11',24],
['21 BBY','The Clone Wars - The Gungan General','1','12',24],
['21 BBY','The Clone Wars - Jedi Crash','1','13',24],
['21 BBY','The Clone Wars - Defenders of Peace','1','14',24],
['21 BBY','The Clone Wars - Trespass','1','15',24],
['21 BBY','The Clone Wars - Blue Shadow Virus','1','17',24],
['21 BBY','The Clone Wars - Mystery of a Thousand Moons','1','18',24],
['21 BBY','The Clone Wars - Storm Over Ryloth','1','19',24],
['21 BBY','The Clone Wars - Innocents of Ryloth','1','20',24],
['21 BBY','The Clone Wars - Liberty on Ryloth','1','21',24],
['21 BBY','The Clone Wars - Holocron Heist','2','1',24],
['21 BBY','The Clone Wars - Cargo of Doom','2','2',24],
['21 BBY','The Clone Wars - Children of the Force','2','3',24],
['21 BBY','The Clone Wars - Bounty Hunters','2','17',24],
['21 BBY','The Clone Wars - The Zillo Beast','2','18',24],
['21 BBY','The Clone Wars - The Zillo Beast Strikes Back','2','19',24],
['21 BBY','The Clone Wars - Senate Spy','2','4',24],
['21 BBY','The Clone Wars - Landing at Point Rain','2','5',24],
['21 BBY','The Clone Wars - Weapons Factory','2','6',24],
['21 BBY','The Clone Wars - Legacy of Terror','2','7',24],
['21 BBY','The Clone Wars - Brain Invaders','2','8',24],
['21 BBY','The Clone Wars - Grievous Intrigue','2','9',24],
['21 BBY','The Clone Wars - The Deserter','2','10',24],
['21 BBY','The Clone Wars - Lightsaber Lost','2','11',24],
['21 BBY','The Clone Wars - The Mandalore Plot','2','12',24],
['21 BBY','The Clone Wars - Voyage of Temptation','2','13',24],
['21 BBY','The Clone Wars - Duchess of Mandalore','2','14',24],
['21 BBY','The Clone Wars - Death Trap','2','20',24],
['21 BBY','The Clone Wars - R2 Come Home','2','21',24],
['21 BBY','The Clone Wars - Lethal Trackdown','2','22',24],
['21 BBY','The Clone Wars - Corruption','3','5',24],
['21 BBY','The Clone Wars - The Academy','3','6',24],
['21 BBY','The Clone Wars - Assassin','3','7',24],
['21 BBY','The Clone Wars - ARC Troopers','3','2',24],
['21 BBY','The Clone Wars - Sphere of Influence','3','4',24],
['21 BBY','The Clone Wars - Evil Plans','3','8',24],
['21 BBY','The Clone Wars - Hostage Crisis','1','22',24],
['21 BBY','The Clone Wars - Hunt for Ziro','3','9',24],
['21 BBY','The Clone Wars - Heroes on Both Sides','3','10',24],
['21 BBY','The Clone Wars - Pursuit of Peace','3','11',24],
['21 BBY','The Clone Wars - Senate Murders','2','15',24],
['20 BBY','The Clone Wars - Nightsisters','3','12',24],
['20 BBY','The Clone Wars - Monster','3','13',24],
['20 BBY','The Clone Wars - Witches of the Mist','3','14',24],
['20 BBY','The Clone Wars - Overlords','3','15',24],
['20 BBY','The Clone Wars - Altar of Mortis','3','16',24],
['20 BBY','The Clone Wars - Ghosts of Mortis','3','17',24],
['20 BBY','The Clone Wars - The Citadel','3','18',24],
['20 BBY','The Clone Wars - Counterattack','3','19',24],
['20 BBY','The Clone Wars - Citadel Rescue','3','20',24],
['20 BBY','The Clone Wars - Padawan Lost','3','21',24],
['20 BBY','The Clone Wars - Wookiee Hunt','3','22',24],
['20 BBY','The Clone Wars - Water War','4','1',24],
['20 BBY','The Clone Wars - Gungan Attack','4','2',24],
['20 BBY','The Clone Wars - Prisoners','4','3',24],
['20 BBY','The Clone Wars - Shadow Warrior','4','4',24],
['20 BBY','The Clone Wars - Mercy Mission','4','5',24],
['20 BBY','The Clone Wars - Nomad Droids','4','6',24],
['20 BBY','The Clone Wars - Darkness on Umbara','4','7',24],
['20 BBY','The Clone Wars - The General','4','8',24],
['20 BBY','The Clone Wars - Plan of Dissent','4','9',24],
['20 BBY','The Clone Wars - Carnage of Krell','4','10',24],
['20 BBY','The Clone Wars - Kidnapped','4','11',24],
['20 BBY','The Clone Wars - Slaves of the Republic','4','12',24],
['20 BBY','The Clone Wars - Escape from Kadavo','4','13',24],
['20 BBY','The Clone Wars - A Friend in Need','4','14',24],
['20 BBY','The Clone Wars - Deception','4','15',24],
['20 BBY','The Clone Wars - Friends and Enemies','4','16',24],
['20 BBY','The Clone Wars - The Box','4','17',24],
['20 BBY','The Clone Wars - Crisis on Naboo','4','18',24],
['20 BBY','The Clone Wars - Massacre','4','19',24],
['20 BBY','Tales of the Empire - The Path of Fear','1','1',20],
['20 BBY','The Clone Wars - Bounty','4','20',24],
['20 BBY','The Clone Wars - Brothers','4','21',24],
['20 BBY','The Clone Wars - Revenge','4','22',24],
['20 BBY','The Clone Wars - A War on Two Fronts','5','2',24],
['20 BBY','The Clone Wars - Front Runners','5','3',24],
['20 BBY','The Clone Wars - The Soft War','5','4',24],
['20 BBY','The Clone Wars - Tipping Points','5','5',24],
['20 BBY','The Clone Wars - The Gathering','5','6',24],
['20 BBY','The Clone Wars - A Test of Strength','5','7',24],
['20 BBY','The Clone Wars - Bound for Rescue','5','8',24],
['20 BBY','The Clone Wars - A Necessary Bond','5','9',24],
['20 BBY','The Clone Wars - Secret Weapons','5','10',24],
['20 BBY','The Clone Wars - A Sunny Day in the Void','5','11',24],
['20 BBY','The Clone Wars - Missing in Action','5','12',24],
['20 BBY','The Clone Wars - Point of No Return','5','13',24],
['19 BBY','The Clone Wars - Revival','5','1',24],
['19 BBY','The Clone Wars - Eminence','5','14',24],
['19 BBY','The Clone Wars - Shades of Reason','5','15',24],
['19 BBY','The Clone Wars - The Lawless','5','16',24],
['19 BBY','The Clone Wars - Sabotage','5','17',24],
['19 BBY','The Clone Wars - The Jedi Who Knew Too Much','5','18',24],
['19 BBY','The Clone Wars - To Catch a Jedi','5','19',24],
['19 BBY','The Clone Wars - The Wrong Jedi','5','20',24],
['19 BBY','The Clone Wars - The Unknown','6','1',24],
['19 BBY','The Clone Wars - Conspiracy','6','2',24],
['19 BBY','The Clone Wars - Fugitive','6','3',24],
['19 BBY','The Clone Wars - Orders','6','4',24],
['19 BBY','The Clone Wars - An Old Friend','6','5',24],
['19 BBY','The Clone Wars - The Rise of Clovis','6','6',24],
['19 BBY','The Clone Wars - Crisis at the Heart','6','7',24],
['19 BBY','The Clone Wars - The Disappeared Part I','6','8',24],
['19 BBY','The Clone Wars - The Disappeared Part II','6','9',24],
['19 BBY','The Clone Wars - The Lost One','6','10',24],
['19 BBY','The Clone Wars - Voices','6','11',24],
['19 BBY','The Clone Wars - Destiny','6','12',24],
['19 BBY','The Clone Wars - Sacrifice','6','13',24],
['19 BBY','The Clone Wars - Gone with a Trace','7','5',24],
['19 BBY','The Clone Wars - Deal No Deal','7','6',24],
['19 BBY','The Clone Wars - Dangerous Debt','7','7',24],
['19 BBY','The Clone Wars - Together Again','7','8',24],
['19 BBY','The Clone Wars - The Bad Batch','7','1',30],
['19 BBY','The Clone Wars - A Distant Echo','7','2',24],
['19 BBY','The Clone Wars - On the Wings of Keeradaks','7','3',24],
['19 BBY','The Clone Wars - Unfinished Business','7','4',24],
['19 BBY','The Clone Wars - Old Friends Not Forgotten','7','9',24],
['19 BBY','Episode III Revenge of the Sith','','',140],
['19 BBY','The Clone Wars - The Phantom Apprentice','7','10',24],
['19 BBY','The Clone Wars - Shattered','7','11',24],
['19 BBY','The Bad Batch - Aftermath','1','1',30],
['19 BBY','The Clone Wars - Victory and Death','7','12',24],
['','Tales of the Empire - Devoted','1','2',20],
['','Tales of the Jedi - Resolve','1','6',20],
['19 BBY','The Bad Batch - Cut and Run','1','2',30],
['19 BBY','The Bad Batch - Replacements','1','3',30],
['19 BBY','The Bad Batch - Cornered','1','4',30],
['19 BBY','The Bad Batch - Rampage','1','5',30],
['19 BBY','The Bad Batch - Decommissioned','1','6',30],
['19 BBY','The Bad Batch - Battle Scars','1','7',30],
['19 BBY','The Bad Batch - Reunion','1','8',30],
['19 BBY','The Bad Batch - Bounty Lost','1','9',30],
['19 BBY','The Bad Batch - Common Ground','1','10',30],
['19 BBY','The Bad Batch - Devil\'s Deal','1','11',30],
['19 BBY','The Bad Batch - Rescue on Ryloth','1','12',30],
['19 BBY','The Bad Batch - Infested','1','13',30],
['19 BBY','The Bad Batch - War-Mantle','1','14',30],
['19 BBY','The Bad Batch - Return to Kamino','1','15',30],
['19 BBY','The Bad Batch - Kamino Lost','1','16',30],
['19–18 BBY','The Bad Batch - Spoils of War','2','1',30],
['19–18 BBY','The Bad Batch - Ruins of War','2','2',30],
['19–18 BBY','The Bad Batch - The Solitary Clone','2','3',30],
['19–18 BBY','The Bad Batch - Faster','2','4',30],
['19–18 BBY','The Bad Batch - Entombed','2','5',30],
['19–18 BBY','The Bad Batch - Tribe','2','6',30],
['18 BBY','The Bad Batch - The Clone Conspiracy','2','7',30],
['18 BBY','The Bad Batch - Truth and Consequences','2','8',30],
['','Tales of the Underworld - A Way Forward','1','4',20],
['','Tales of the Underworld - Friends','1','5',20],
['','Tales of the Underworld - One Warrior to Another','1','6',20],
['18 BBY','The Bad Batch - The Crossing','2','9',30],
['18 BBY','The Bad Batch - Retrieval','2','10',30],
['18 BBY','The Bad Batch - Metamorphosis','2','11',30],
['18 BBY','The Bad Batch - The Outpost','2','12',30],
['18 BBY','The Bad Batch - Pabu','2','13',30],
['18 BBY','The Bad Batch - Tipping Point','2','14',30],
['18 BBY','The Bad Batch - The Summit','2','15',30],
['18 BBY','The Bad Batch - Plan 99','2','16',30],
['19 BBY','The Bad Batch - Confined','3','1',30],
['20 BBY','The Bad Batch - Paths Unknown','3','2',30],
['21 BBY','The Bad Batch - Shadows of Tantiss','3','3',30],
['22 BBY','The Bad Batch - A Different Approach','3','4',30],
['23 BBY','The Bad Batch - The Return','3','5',30],
['24 BBY','The Bad Batch - Infiltration','3','6',30],
['25 BBY','The Bad Batch - Extraction','3','7',30],
['26 BBY','The Bad Batch - Bad Territory','3','8',30],
['27 BBY','The Bad Batch - The Harbinger','3','9',30],
['28 BBY','The Bad Batch - Identity Crisis','3','10',30],
['29 BBY','The Bad Batch - Point of No Return','3','11',30],
['30 BBY','The Bad Batch - Juggernaut','3','12',30],
['31 BBY','The Bad Batch - Into the Breach','3','13',30],
['32 BBY','The Bad Batch - Flash Strike','3','14',30],
['33 BBY','The Bad Batch - The Cavalry Has Arrived','3','15',30],
['34 BBY','Tales of the Empire - Realization','1','3',20],
['10 BBY','Solo A Story','','',135],
['c. 9 BBY–c. 2 BBY','Tales of the Empire - The Path of Anger','1','4',20],
['9 BBY','Obi-Wan Kenobi - Part I','1','1',47],
['9 BBY','Obi-Wan Kenobi - Part II','1','2',47],
['9 BBY','Obi-Wan Kenobi - Part III','1','3',47],
['9 BBY','Obi-Wan Kenobi - Part IV','1','4',47],
['9 BBY','Obi-Wan Kenobi - Part V','1','5',47],
['9 BBY','Obi-Wan Kenobi - Part VI','1','6',47],
['5 BBY','Andor - Kassa','1','1',50],
['5 BBY','Andor - That Would Be Me','1','2',50],
['5 BBY','Andor - Reckoning','1','3',50],
['5 BBY','Andor - Aldhani','1','4',50],
['5 BBY','Andor - The Axe Forgets','1','5',50],
['5 BBY','Andor - The Eye','1','6',50],
['5 BBY','Andor - Announcement','1','7',50],
['5 BBY','Andor - Narkina 5','1','8',50],
['5 BBY','Andor - Nobody\'s Listening!','1','9',50],
['5 BBY','Andor - One Way Out','1','10',50],
['5 BBY','Andor - Daughter of Ferrix','1','11',50],
['5 BBY','Andor - Rix Road','1','12',50],
['5 BBY','Rebels - Spark of Rebellion','1','1',44],
['5 BBY','Rebels - Droids in Distress','1','3',22],
['5 BBY','Rebels - Fighter Flight','1','4',22],
['5 BBY','Rebels - Rise of the Old Masters','1','5',22],
['5 BBY','Rebels - Breaking Ranks','1','6',22],
['5 BBY','Rebels - Out of Darkness','1','7',22],
['4 BBY','Rebels - Empire Day','1','8',22],
['4 BBY','Rebels - Gathering Forces','1','9',22],
['4 BBY','Rebels - Path of the Jedi','1','10',22],
['4 BBY','Rebels - Idiot\'s Array','1','11',22],
['4 BBY','Rebels - Vision of Hope','1','12',22],
['4 BBY','Rebels - Call to Action','1','13',22],
['4 BBY','Rebels - Rebel Resolve','1','14',22],
['4 BBY','Rebels - Fire Across the Galaxy','1','15',22],
['4 BBY','Andor - One Year Later','2','1',50],
['4 BBY','Andor - Sagrona Teema','2','2',50],
['4 BBY','Andor - Harvest','2','3',50],
['4 BBY','Rebels - The Siege of Lothal','2','1',44],
['4 BBY','Rebels - The Lost Commanders','2','3',22],
['4 BBY','Rebels - Relics of the Old Republic','2','4',22],
['4 BBY','Rebels - Always Two There Are','2','5',22],
['4 BBY','Rebels - Brothers of the Broken Horn','2','6',22],
['4 BBY','Rebels - Wings of the Master','2','7',22],
['4 BBY','Rebels - Blood Sisters','2','8',22],
['4 BBY','Rebels - Stealth Strike','2','9',22],
['3 BBY','Rebels - The Future of the Force','2','10',22],
['3 BBY','Andor - Ever Been to Ghorman?','2','4',50],
['3 BBY','Andor - I Have Friends Everywhere','2','5',50],
['3 BBY','Andor - What a Festive Evening','2','6',50],
['3 BBY','Rebels - Legacy','2','11',22],
['3 BBY','Rebels - A Princess on Lothal','2','12',22],
['3 BBY','Rebels - The Protector of Concord Dawn','2','13',22],
['3 BBY','Rebels - Legends of the Lasat','2','14',22],
['3 BBY','Rebels - The Call','2','15',22],
['3 BBY','Rebels - Homecoming','2','16',22],
['3 BBY','Rebels - The Honorable Ones','2','17',22],
['3 BBY','Rebels - Shroud of Darkness','2','18',22],
['3 BBY','Rebels - The Forgotten Droid','2','19',22],
['3 BBY','Rebels - The Mystery of Chopper Base','2','20',22],
['3 BBY','Rebels - Twilight of the Apprentice','2','21',44],
['2 BBY','Rebels - Steps Into Shadow','3','1',44],
['2 BBY','Rebels - The Holocrons of Fate','3','3',22],
['2 BBY','Rebels - The Antilles Extraction','3','4',22],
['2 BBY','Rebels - Hera\'s Heroes','3','5',22],
['2 BBY','Rebels - The Last Battle','3','6',22],
['2 BBY','Rebels - Imperial Supercommandos','3','7',22],
['2 BBY','Rebels - Iron Squadron','3','8',22],
['2 BBY','Rebels - The Wynkahthu Job','3','9',22],
['2 BBY','Rebels - An Inside Man','3','10',22],
['2 BBY','Rebels - Visions and Voices','3','11',22],
['2 BBY','Rebels - Ghosts of Geonosis','3','12',44],
['2 BBY','Rebels - Warhead','3','14',22],
['2 BBY','Rebels - Trials of the Darksaber','3','15',22],
['2 BBY','Rebels - Legacy of Mandalore','3','16',22],
['2 BBY','Andor - Messenger','2','7',50],
['2 BBY','Andor - Who Are You?','2','8',50],
['2 BBY','Rebels - Through Imperial Eyes','3','17',22],
['2 BBY','Andor - Welcome to the Rebellion','2','9',50],
['2 BBY','Rebels - Secret Cargo','3','18',22],
['2 BBY','Rebels - Double Agent Droid','3','19',22],
['2 BBY','Rebels - Twin Suns','3','20',22],
['2 BBY','Rebels - Zero Hour','3','21',44],
['1 BBY','Rebels - Heroes of Mandalore','4','1',22],
['1 BBY','Rebels - In the Name of the Rebellion','4','3',44],
['1 BBY','Rebels - The Occupation','4','5',22],
['1 BBY','Rebels - Flight of the Defender','4','6',22],
['1 BBY','Rebels - Kindred','4','7',22],
['1 BBY','Rebels - Crawler Commandeers','4','8',22],
['1 BBY','Rebels - Rebel Assault','4','9',22],
['1 BBY','Rebels - Jedi Night','4','10',22],
['1 BBY','Rebels - DUME','4','11',22],
['1 BBY','Rebels - Wolves and a Door','4','12',22],
['1 BBY','Rebels - A World Between Worlds','4','13',22],
['1 BBY','Rebels - A Fool\'s Hope','4','14',22],
['1 BBY','Rebels - Family Reunion and Farewell','4','15',22],
['1 BBY','Tales of the Empire - The Way Out','1','5',20],
['1 BBY','Andor - Make It Stop','2','10',50],
['1 BBY','Andor - Who Else Knows?','2','11',50],
['1 BBY','Andor - Jedha Kyber Erso','2','12',50],
['1 BBY','Rogue One A Story','','',133],
['1 BBY','Episode IV A New Hope','','',121],
['3 ABY','Episode V The Empire Strikes Back','','',124],
['4 ABY','Episode VI Return of the Jedi','','',131],
['9 ABY','The Mandalorian - Chapter 1 The Mandalorian','1','1',40],
['9 ABY','The Mandalorian - Chapter 2 The Child','1','2',40],
['9 ABY','The Mandalorian - Chapter 3 The Sin','1','3',40],
['9 ABY','The Mandalorian - Chapter 4 Sanctuary','1','4',40],
['9 ABY','The Mandalorian - Chapter 5 The Gunslinger','1','5',40],
['9 ABY','The Mandalorian - Chapter 6 The Prisoner','1','6',40],
['9 ABY','The Mandalorian - Chapter 7 The Reckoning','1','7',40],
['9 ABY','The Mandalorian - Chapter 8 Redemption','1','8',40],
['9 ABY','The Mandalorian - Chapter 9 The Marshal','2','1',40],
['9 ABY','The Mandalorian - Chapter 10 The Passenger','2','2',40],
['','Tales of the Empire - The Path of Hate','1','6',20],
['9 ABY','The Mandalorian - Chapter 11 The Heiress','2','3',40],
['9 ABY','The Mandalorian - Chapter 12 The Siege','2','4',40],
['9 ABY','The Mandalorian - Chapter 13 The Jedi','2','5',40],
['9 ABY','The Mandalorian - Chapter 14 The Tragedy','2','6',40],
['9 ABY','The Mandalorian - Chapter 15 The Believer','2','7',40],
['9 ABY','The Mandalorian - Chapter 16 The Rescue','2','8',40],
['9 ABY','The Book of Boba Fett - Chapter 1 Stranger in a Strange Land','1','1',55],
['9 ABY','The Book of Boba Fett - Chapter 2 The Tribes of Tatooine','1','2',55],
['9 ABY','The Book of Boba Fett - Chapter 3 The Streets of Mos Espa','1','3',55],
['9 ABY','The Book of Boba Fett - Chapter 4 The Gathering Storm','1','4',55],
['c. 9 ABY','The Book of Boba Fett - Chapter 5 Return of the Mandalorian','1','5',55],
['c. 9 ABY','The Book of Boba Fett - Chapter 6 From the Desert Comes a Stranger','1','6',55],
['c. 9 ABY','The Book of Boba Fett - Chapter 7 In the Name of Honor','1','7',55],
['9 ABY','The Mandalorian - Chapter 17 The Apostate','3','1',40],
['9 ABY','The Mandalorian - Chapter 18 The Mines of Mandalore','3','2',40],
['9 ABY','The Mandalorian - Chapter 19 The Convert','3','3',40],
['9 ABY','The Mandalorian - Chapter 20 The Foundling','3','4',40],
['9 ABY','The Mandalorian - Chapter 21 The Pirate','3','5',40],
['9 ABY','The Mandalorian - Chapter 22 Guns for Hire','3','6',40],
['9 ABY','The Mandalorian - Chapter 23 The Spies','3','7',40],
['9 ABY','The Mandalorian - Chapter 24 The Return','3','8',40],
['c. 9 ABY','Ahsoka - Part One Master and Apprentice','1','1',47],
['c. 9 ABY','Ahsoka - Part Two Toil and Trouble','1','2',47],
['c. 9 ABY','Ahsoka - Part Three Time to Fly','1','3',47],
['c. 9 ABY','Ahsoka - Part Four Fallen Jedi','1','4',47],
['c. 9 ABY','Ahsoka - Part Five Shadow Warrior','1','5',47],
['c. 9 ABY','Ahsoka - Part Six Far Far Away','1','6',47],
['c. 9 ABY','Ahsoka - Part Seven Dreams and Madness','1','7',47],
['c. 9 ABY','Ahsoka - Part Eight The Jedi the Witch and the Warlord','1','8',47],
['c. 9 ABY','Skeleton Crew - This Could Be a Real Adventure','1','1',42],
['c. 9 ABY','Skeleton Crew - Way Way Out Past the Barrier','1','2',42],
['c. 9 ABY','Skeleton Crew - Very Interesting As an Astrogation Problem','1','3',42],
['c. 9 ABY','Skeleton Crew - Can\'t Say I Remember No At Attin','1','4',42],
['c. 9 ABY','Skeleton Crew - You Have a Lot to Learn About Pirates','1','5',42],
['c. 9 ABY','Skeleton Crew - Zero Friends Again','1','6',42],
['c. 9 ABY','Skeleton Crew - We\'re Gonna Be In So Much Trouble','1','7',42],
['c. 9 ABY','Skeleton Crew - The Real Good Guys','1','8',42],
['c. 9 ABY','The Mandalorian and Grogu','','',132],
['34 ABY','Episode VII The Force Awakens','','',138],
['34 ABY','Episode VIII The Last Jedi','','',158],
['35 ABY','Episode IX The Rise of Skywalker','','',141],
].map(([Year,NAME,S,E,Duration])=>({Year,NAME,S,E,Duration}));

function makeKey(r){return`${r.Year}|${r.NAME}|${r.S}|${r.E}`}
function saveDone(){localStorage.setItem(KEY,JSON.stringify(doneMap))}
function formatMinutes(m){const d=Math.floor(m/1440);m%=1440;const h=Math.floor(m/60),mm=m%60;return`${d}d ${h}h ${mm}m`}
function formatHM(m){const h=Math.floor(m/60),mm=m%60;return`${h}h ${mm}m`}

// ── Shared film keywords (fix: single source of truth, no duplication) ───────
const FILM_KEYWORDS=["episode","phantom menace","attack of the clones","revenge of the sith",
  "solo","rogue one","a new hope","empire strikes back","return of the jedi",
  "force awakens","last jedi","rise of skywalker"];

function isFilmEntry(name){
  const n=name.toLowerCase();
  return FILM_KEYWORDS.some(k=>n.includes(k));
}

// ── Single-pass stats (fix: compute once, reuse everywhere) ──────────────────
function getStats(){
  let watched=0,remaining=0,totalMin=0,doneCount=0,totalCount=0;
  rows.forEach(r=>{
    const k=makeKey(r),e=doneMap[k];
    totalCount++;
    totalMin+=r.Duration;
    if(e&&e.done){watched+=r.Duration;doneCount++;}
    else{remaining+=r.Duration;}
  });
  return{watched,remaining,totalMin,doneCount,totalCount};
}

// Same but scoped to filtered rows (for render header stats)
function getFilteredStats(filter){
  const f=filter?filter.toLowerCase():"";
  let total=0,done=0,totalMin=0,doneMin=0;
  rows.forEach(r=>{
    const m=!f||r.NAME.toLowerCase().includes(f)||r.Year.toLowerCase().includes(f);
    if(!m)return;
    const e=doneMap[makeKey(r)];
    total++;totalMin+=r.Duration;
    if(e&&e.done){done++;doneMin+=r.Duration;}
  });
  return{total,done,totalMin,doneMin};
}

function getWatchedToday(){
  const{watched}=getStats();
  return Math.max(0,watched-previousMinutes);
}

function resetDayStats(){
  const{watched}=getStats();
  const w=Math.max(0,watched-previousMinutes);
  dayHistory.push({date:new Date().toISOString().split("T")[0],minutes:w});
  localStorage.setItem(DAY_HISTORY_KEY,JSON.stringify(dayHistory));
  previousMinutes=watched;
  localStorage.setItem(PREV_MIN_KEY,previousMinutes);
  dayReset=Date.now();
  localStorage.setItem(DAY_KEY,dayReset);
  updateAllStats();
}
document.getElementById("resetDayBtn").addEventListener("click",resetDayStats);

function getFranchiseColor(n){
  n=n.toLowerCase();
  if(n.includes("tales of"))return"#992020";
  if(isFilmEntry(n))return"#D12A2A";  // fix: reuse shared helper, no duplication
  if(n.includes("clone wars"))return"#FF8F2A";
  if(n.includes("bad batch"))return"#FFDF4A";
  if(n.includes("obi-wan")||n.includes("obi wan"))return"#8FEF78";
  if(n.includes("andor"))return"#1C7F3E";
  if(n.includes("rebels"))return"#6FC2FF";
  if(n.includes("mandalorian"))return"#0A4A7F";
  if(n.includes("ahsoka"))return"#CFCFCF";
  if(n.includes("skeleton crew"))return"#3E3E3E";
  return"#07101f";
}

function isLightColor(h){
  h=h.replace("#","");
  const r=parseInt(h.substring(0,2),16),
        g=parseInt(h.substring(2,4),16),
        b=parseInt(h.substring(4,6),16),
        L=.299*r+.587*g+.114*b;
  return L>160;
}

// ── Render: only rebuilds DOM when filter changes ─────────────────────────────
let currentFilter=null; // null = never rendered, forces first build
let renderedRowIndices=[];

function render(f=""){
  if(f!==currentFilter){
    currentFilter=f;
    buildCards(f);
  }
  updateStats(f);
}

function buildCards(f){
  elCardContainer.innerHTML="";
  renderedRowIndices=[];
  const fl=f?f.toLowerCase():"";

  rows.forEach((r,i)=>{
    const m=!fl||r.NAME.toLowerCase().includes(fl)||r.Year.toLowerCase().includes(fl);
    if(!m)return;

    renderedRowIndices.push(i);
    const e=doneMap[makeKey(r)];
    const bg=getFranchiseColor(r.NAME),
          light=isLightColor(bg),
          tc=light?"#000":"#fff",
          mc=light?"#222":"#9fb4ff";
    const film=isFilmEntry(r.NAME);  // fix: shared helper

    let meta=`<b>${r.Year}</b>`;
    if(!film)meta+=` • S${r.S||"-"}E${r.E||"-"}`;
    meta+=` • <span class="badge">${film?"Film":"Series"}</span>`;

    const card=document.createElement("div");
    card.className="entry-card"+(e&&e.done?" done":"");
    card.style.cssText=`background:${bg};border-color:${bg}`;
    card.innerHTML=`<div class="entry-title" style="color:${tc};">${r.NAME}</div><div class="entry-meta" style="color:${mc};">${meta}</div>`;
    card.dataset.rowIndex=i;

    elCardContainer.appendChild(card);
  });

  // Single delegated click listener (fix: one listener vs hundreds)
  elCardContainer.onclick=e=>{
    const card=e.target.closest(".entry-card");
    if(!card)return;
    const i=Number(card.dataset.rowIndex);
    const k=makeKey(rows[i]),entry=doneMap[k];

    if(!entry||!entry.done){
      for(let j=0;j<=i;j++){doneMap[makeKey(rows[j])]={done:true,ts:Date.now()};}
    }else{
      for(let j=i;j<rows.length;j++){delete doneMap[makeKey(rows[j])];}
    }
    saveDone();
    refreshCardStates();
    updateStats(currentFilter);
    updateAllStats();
    scrollToFirstUnchecked();
  };
}

function refreshCardStates(){
  elCardContainer.querySelectorAll(".entry-card").forEach(card=>{
    const i=Number(card.dataset.rowIndex);
    const e=doneMap[makeKey(rows[i])];
    card.classList.toggle("done",!!(e&&e.done));
  });
}

// ── Scroll to first unchecked ─────────────────────────────────────────────────
function scrollToFirstUnchecked(){
  const firstUnchecked=elCardContainer.querySelector(".entry-card:not(.done)");
  if(firstUnchecked){
    setTimeout(()=>{
      const top=firstUnchecked.getBoundingClientRect().top+window.scrollY;
      const stickyH=elStickyTop.offsetHeight+elStickyControls.offsetHeight;
      window.scrollTo({top:top-370,behavior:"smooth"});
    },50);
  }
}

// ── Update summary line stats (no card rebuild) ───────────────────────────────
function updateStats(f=""){
  const fl=f?f.toLowerCase():"";
  const{total,done,totalMin,doneMin}=getFilteredStats(fl);
  }

function _updateTotalProgress(){
  const { watched, totalMin } = getStats();
  const pct = totalMin ? (watched / totalMin) * 100 : 0;
  elTotalProgressFill.style.width = Math.min(pct,100) + "%";
  elTotalProgressStatus.textContent =
    `${formatMinutes(watched)} / ${formatMinutes(totalMin)} (${pct.toFixed(1)}%)`;
}


function updateAllStats(){
  const{watched}=getStats();
  const watchedToday=Math.max(0,watched-previousMinutes);
  _updateTotalProgress();
  _updateDailyGoal(watchedToday);
  _updateCurrentPace(watched);
}

// ── Live countdown to end date (replaces projected finish) ────────────────────
let countdownInterval=null;
function startLiveCountdown(){
  if(countdownInterval)clearInterval(countdownInterval);
  const v=elEndDate.value;
  if(!v){elLiveCountdown.textContent="";return;}
  const end=new Date(v);
  end.setHours(23,59,59,999); // count to end of the target day

  function tick(){
    const diff=end-Date.now();
    if(diff<=0){elLiveCountdown.textContent="⏰ Time's up!";clearInterval(countdownInterval);return;}
    const d=Math.floor(diff/86400000),
          h=Math.floor(diff/3600000%24),
          m=Math.floor(diff/60000%60),
          s=Math.floor(diff/1000%60);
    elLiveCountdown.textContent=`⏳ ${d}d ${h}h ${m}m ${s}s remaining `;
  }
  tick();
  countdownInterval=setInterval(tick,1000);
}

function calculateBingeGoal(){
  const v=elEndDate.value;
  if(!v)return;
  const end=new Date(v),
        today=new Date();
  today.setHours(0,0,0,0);
  const diff=Math.ceil((end-today)/86400000);
  if(diff<=0)return;

  const{remaining}=getStats();
  const per=remaining/diff;
  lastPerDayTarget=per;
  updateAllStats();
}

// Internal update functions accept pre-computed values (fix: no per-function row scans)
function _updateDailyGoal(watchedToday){
  const per=lastPerDayTarget;
  if(!per||per<=0)return;
  const pct=Math.min(watchedToday/per*100,100);
  elDailyGoalFill.style.width=pct+"%";
  let status="";
  if(watchedToday>=per){status="🔥 Ahead of schedule";elDailyGoalFill.style.background="#4caf50";}
  else if(watchedToday>=per*.75){status="⚡ On pace";elDailyGoalFill.style.background="#ffc107";}
  else{status="⏳ Behind schedule";elDailyGoalFill.style.background="#f44336";}
  elDailyGoalStatus.innerHTML=`${status} • ${formatHM(watchedToday)} / ${formatHM(Math.ceil(per))}`;
}

function _updateCurrentPace(watched){
  const startVal=elStartDate.value;
  if(!startVal){elCurrentPace.textContent="";return;}
  const start=new Date(startVal),today=new Date();
  today.setHours(0,0,0,0);
  const daysPassed=Math.max(1,Math.floor((today-start)/86400000));
  const avg=watched/daysPassed;
  elCurrentPace.textContent=`Current Pace: ${formatHM(Math.ceil(avg))}/day`;
}

// Public wrappers
function updateDailyGoal(per){lastPerDayTarget=per;const{watched}=getStats();_updateDailyGoal(Math.max(0,watched-previousMinutes));}
function updateCurrentPace(){const{watched}=getStats();_updateCurrentPace(watched);}


elSearchInput.addEventListener("input",e=>{render(e.target.value)});

document.getElementById("clearDoneBtn").addEventListener("click",()=>{
  doneMap={};
  dayHistory=[];
  previousMinutes=0;
  dayReset=Date.now();
  localStorage.setItem(KEY,"{}");
  localStorage.setItem(DAY_HISTORY_KEY,"[]");
  localStorage.setItem(PREV_MIN_KEY,"0");
  localStorage.setItem(DAY_KEY,dayReset);
  // fix: rebuild cards (state changed) then update stats
  buildCards(currentFilter);
  updateStats(currentFilter);
  updateAllStats();
  scrollToFirstUnchecked();
});

document.getElementById("calcBtn").addEventListener("click",()=>{
  localStorage.setItem("sw_start_date",elStartDate.value);
  localStorage.setItem("sw_end_date",elEndDate.value);
  calculateBingeGoal();
  startLiveCountdown();
});

// Load saved dates on startup
const savedStart=localStorage.getItem("sw_start_date");
const savedEnd=localStorage.getItem("sw_end_date");
if(savedStart)elStartDate.value=savedStart;
if(savedEnd)elEndDate.value=savedEnd;

// Initial render — currentFilter is null so buildCards always runs here
render();
scrollToFirstUnchecked();
if(savedStart&&savedEnd){
  calculateBingeGoal();
}
startLiveCountdown();
updateAllStats();
</script>
</body>
</html>

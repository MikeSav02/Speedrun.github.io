<html lang="en">
<head>
<meta charset="UTF-8">
<title>Star Wars Speedrun Challenge</title>
<base href="./">
<style>
body{font-family:"Segoe UI","Roboto","Helvetica Neue",Arial,sans-serif;background:#050814;color:#e8ecff;margin:0;overflow-x:hidden}
body::before{content:"";position:fixed;inset:0;background-image:linear-gradient(#ff8c4222 1px,transparent 1px),linear-gradient(90deg,#ff8c4222 1px,transparent 1px);background-size:80px 80px;opacity:.25;animation:gridDrift 60s linear infinite;z-index:-1}
@keyframes gridDrift{0%{transform:translate3d(0,0,0)}100%{transform:translate3d(-80px,-80px,0)}}

#stickyTop{position:sticky;top:0;z-index:9999;background:#050814;box-shadow:0 4px 12px #00000088}
header{background:#07101f;padding:18px 22px;border-bottom:2px solid #13243d}
header h1{margin:0;font-size:22px;letter-spacing:.08em;color:#ff8c42;text-shadow:0 0 8px #ff8c4244;text-transform:uppercase}
header .stats{margin-top:6px;font-size:14px;color:#4fc3f7}

#progressContainer{width:100%;background:#0b1628;height:20px;border-radius:6px;margin-top:12px;overflow:hidden;border:1px solid #1f2a44}
#progressBar{height:100%;width:0;background:#ff8c42;transition:width .3s ease;position:relative}
#progressBar span{position:absolute;width:100%;text-align:center;font-family:Consolas,monospace;font-size:13px;font-weight:bold;color:#050814;top:0;line-height:20px}

#stickyControls{background:#050814;padding:12px 22px;border-bottom:1px solid #13243d;display:flex;flex-wrap:wrap;gap:12px;align-items:center;position:sticky;top:110px;z-index:9998}
#stickyControls input[type=text],#stickyControls input[type=date]{padding:8px 10px;font-size:14px;border-radius:4px;border:1px solid #29405f;background:#050814;color:#e8ecff}
#stickyControls button{padding:8px 14px;border-radius:4px;background:#07101f;border:1px solid #ff8c42;color:#ff8c42;font-size:14px;cursor:pointer;text-transform:uppercase;letter-spacing:.06em}
#stickyControls button:hover{background:#0c1a2b}

#liveCountdown{color:#4fc3f7;font-size:14px;font-family:Consolas,monospace}

#dailyGoalContainer{margin-top:6px;width:100%}
#dailyGoalContainer>div:first-child{font-size:14px;color:#4fc3f7}
#dailyGoalBar{width:100%;height:18px;background:#0b1628;border-radius:6px;overflow:hidden;border:1px solid #1f2a44;margin-top:4px}
#dailyGoalFill{height:100%;width:0;background:#4caf50;transition:width .3s ease}
#dailyGoalStatus{margin-top:4px;font-size:14px;color:#4fc3f7}

main{padding:18px 22px}
.summary-line{margin-bottom:18px;color:#4fc3f7;font-size:15px}

.entry-card{border-radius:6px;padding:12px 14px;margin-bottom:12px;box-shadow:0 0 8px #0d1324 inset;transition:background .2s ease,transform .1s ease,opacity .2s ease;border:2px solid transparent;position:relative}
.entry-card::before{content:"";position:absolute;inset:4px;border:1px solid rgba(255,140,66,.18);pointer-events:none}
.entry-card.done{opacity:.55;text-decoration:line-through}
.entry-card:active{transform:scale(.98)}
.entry-title{font-size:17px;font-weight:600}
.entry-meta{font-size:14px;margin-top:4px}
.badge{padding:2px 6px;border-radius:4px;font-size:11px;background:#ff8c4215;color:#ffcc80}

/* MOBILE OPTIMIZATION */
@media (max-width:600px){
header{padding:10px 14px}
header h1{font-size:16px;letter-spacing:.05em;white-space:normal;line-height:1.2}
#progressContainer{height:14px;margin-top:6px}
#progressBar span{font-size:10px;line-height:14px}
#stickyControls{padding:8px 12px;gap:6px;top:80px}
#stickyControls input[type=text],#stickyControls input[type=date]{padding:6px 8px;font-size:12px}
#stickyControls button{padding:6px 10px;font-size:12px}
#dailyGoalContainer{margin-top:4px}
#dailyGoalBar{height:12px}
#dailyGoalStatus,#projectedFinish,#currentPace,#daysElapsed,#liveCountdown{font-size:12px}
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
<header>
<h1>STAR WARS SPEEDRUN CHALLENGE</h1>
<div class="stats" id="headerStats">Loading…</div>
<div id="progressContainer"><div id="progressBar"><span id="progressText"></span></div></div>
</header>

<div id="stickyControls">
<input id="searchInput" type="text" placeholder="Search…">
<input id="startDateInput" type="date">
<input id="endDateInput" type="date">
<button id="calcBtn">Set Goal</button>
<button id="resetDayBtn">Reset Day Stats</button>
<button id="clearDoneBtn">Clear All</button>
<div id="liveCountdown"></div>

<div id="dailyGoalContainer">
<div>Daily Goal Progress</div>
<div id="dailyGoalBar"><div id="dailyGoalFill"></div></div>
<div id="dailyGoalStatus"></div>
<div id="projectedFinish" style="margin-top:6px;color:#4fc3f7;font-size:14px;"></div>
<div id="currentPace" style="margin-top:6px;color:#4fc3f7;font-size:14px;"></div>
<div id="daysElapsed" style="margin-top:4px;color:#4fc3f7;font-size:14px;"></div>
</div>
</div>
</div>

<main>
<div class="summary-line" id="summaryLine"></div>
<div id="cardContainer"></div>
</main>
<textarea id="csvData" style="display:none;">
Year,NAME,S,E,Done,Duration
Between c. 68 58 BBY,Tales of the Jedi - Justice,1,1,FALSE,20
,Tales of the Underworld - The Good Life,1,1,FALSE,20
Between c. 50 BBY and 48 BBY,Tales of the Jedi - Choices,1,2,FALSE,20
,Tales of the Underworld - A Good Turn,1,2,FALSE,20
36 35 BBY,Tales of the Jedi - Life and Death,1,3,FALSE,20
,Tales of the Underworld - One Good Deed,1,3,FALSE,20
32 BBY,Episode I The Phantom Menace,,,FALSE,136
32 BBY,Tales of the Jedi - The Sith Lord,1,4,FALSE,20
22 BBY,Episode II Attack of the Clones,,,FALSE,142
22 BBY,The Clone Wars - Cat and Mouse,2,16,FALSE,24
22 BBY,The Clone Wars - The Hidden Enemy,1,16,FALSE,24
22 BBY,The Clone Wars film,,,FALSE,90
22 BBY,The Clone Wars - Clone Cadets,3,1,FALSE,24
22 BBY,Tales of the Jedi - Practice Makes Perfect,1,5,FALSE,20
22 BBY,The Clone Wars - Supply Lines,3,3,FALSE,24
22 BBY,The Clone Wars - Ambush,1,1,FALSE,24
22 BBY,The Clone Wars - Rising Malevolence,1,2,FALSE,24
22 BBY,The Clone Wars - Shadow of Malevolence,1,3,FALSE,24
22 BBY,The Clone Wars - Destroy Malevolence,1,4,FALSE,24
22 BBY,The Clone Wars - Rookies,1,5,FALSE,24
22 BBY,The Clone Wars - Downfall of a Droid,1,6,FALSE,24
22 BBY,The Clone Wars - Duel of the Droids,1,7,FALSE,24
22 BBY,The Clone Wars - Bombad Jedi,1,8,FALSE,24
21 BBY,The Clone Wars - Cloak of Darkness,1,9,FALSE,24
21 BBY,The Clone Wars - Lair of Grievous,1,10,FALSE,24
21 BBY,The Clone Wars - Dooku Captured,1,11,FALSE,24
21 BBY,The Clone Wars - The Gungan General,1,12,FALSE,24
21 BBY,The Clone Wars - Jedi Crash,1,13,FALSE,24
21 BBY,The Clone Wars - Defenders of Peace,1,14,FALSE,24
21 BBY,The Clone Wars - Trespass,1,15,FALSE,24
21 BBY,The Clone Wars - Blue Shadow Virus,1,17,FALSE,24
21 BBY,The Clone Wars - Mystery of a Thousand Moons,1,18,FALSE,24
21 BBY,The Clone Wars - Storm Over Ryloth,1,19,FALSE,24
21 BBY,The Clone Wars - Innocents of Ryloth,1,20,FALSE,24
21 BBY,The Clone Wars - Liberty on Ryloth,1,21,FALSE,24
21 BBY,The Clone Wars - Holocron Heist,2,1,FALSE,24
21 BBY,The Clone Wars - Cargo of Doom,2,2,FALSE,24
21 BBY,The Clone Wars - Children of the Force,2,3,FALSE,24
21 BBY,The Clone Wars - Bounty Hunters,2,17,FALSE,24
21 BBY,The Clone Wars - The Zillo Beast,2,18,FALSE,24
21 BBY,The Clone Wars - The Zillo Beast Strikes Back,2,19,FALSE,24
21 BBY,The Clone Wars - Senate Spy,2,4,FALSE,24
21 BBY,The Clone Wars - Landing at Point Rain,2,5,FALSE,24
21 BBY,The Clone Wars - Weapons Factory,2,6,FALSE,24
21 BBY,The Clone Wars - Legacy of Terror,2,7,FALSE,24
21 BBY,The Clone Wars - Brain Invaders,2,8,FALSE,24
21 BBY,The Clone Wars - Grievous Intrigue,2,9,FALSE,24
21 BBY,The Clone Wars - The Deserter,2,10,FALSE,24
21 BBY,The Clone Wars - Lightsaber Lost,2,11,FALSE,24
21 BBY,The Clone Wars - The Mandalore Plot,2,12,FALSE,24
21 BBY,The Clone Wars - Voyage of Temptation,2,13,FALSE,24
21 BBY,The Clone Wars - Duchess of Mandalore,2,14,FALSE,24
21 BBY,The Clone Wars - Death Trap,2,20,FALSE,24
21 BBY,The Clone Wars - R2 Come Home,2,21,FALSE,24
21 BBY,The Clone Wars - Lethal Trackdown,2,22,FALSE,24
21 BBY,The Clone Wars - Corruption,3,5,FALSE,24
21 BBY,The Clone Wars - The Academy,3,6,FALSE,24
21 BBY,The Clone Wars - Assassin,3,7,FALSE,24
21 BBY,The Clone Wars - ARC Troopers,3,2,FALSE,24
21 BBY,The Clone Wars - Sphere of Influence,3,4,FALSE,24
21 BBY,The Clone Wars - Evil Plans,3,8,FALSE,24
21 BBY,The Clone Wars - Hostage Crisis,1,22,FALSE,24
21 BBY,The Clone Wars - Hunt for Ziro,3,9,FALSE,24
21 BBY,The Clone Wars - Heroes on Both Sides,3,10,FALSE,24
21 BBY,The Clone Wars - Pursuit of Peace,3,11,FALSE,24
21 BBY,The Clone Wars - Senate Murders,2,15,FALSE,24
20 BBY,The Clone Wars - Nightsisters,3,12,FALSE,24
20 BBY,The Clone Wars - Monster,3,13,FALSE,24
20 BBY,The Clone Wars - Witches of the Mist,3,14,FALSE,24
20 BBY,The Clone Wars - Overlords,3,15,FALSE,24
20 BBY,The Clone Wars - Altar of Mortis,3,16,FALSE,24
20 BBY,The Clone Wars - Ghosts of Mortis,3,17,FALSE,24
20 BBY,The Clone Wars - The Citadel,3,18,FALSE,24
20 BBY,The Clone Wars - Counterattack,3,19,FALSE,24
20 BBY,The Clone Wars - Citadel Rescue,3,20,FALSE,24
20 BBY,The Clone Wars - Padawan Lost,3,21,FALSE,24
20 BBY,The Clone Wars - Wookiee Hunt,3,22,FALSE,24
20 BBY,The Clone Wars - Water War,4,1,FALSE,24
20 BBY,The Clone Wars - Gungan Attack,4,2,FALSE,24
20 BBY,The Clone Wars - Prisoners,4,3,FALSE,24
20 BBY,The Clone Wars - Shadow Warrior,4,4,FALSE,24
20 BBY,The Clone Wars - Mercy Mission,4,5,FALSE,24
20 BBY,The Clone Wars - Nomad Droids,4,6,FALSE,24
20 BBY,The Clone Wars - Darkness on Umbara,4,7,FALSE,24
20 BBY,The Clone Wars - The General,4,8,FALSE,24
20 BBY,The Clone Wars - Plan of Dissent,4,9,FALSE,24
20 BBY,The Clone Wars - Carnage of Krell,4,10,FALSE,24
20 BBY,The Clone Wars - Kidnapped,4,11,FALSE,24
20 BBY,The Clone Wars - Slaves of the Republic,4,12,FALSE,24
20 BBY,The Clone Wars - Escape from Kadavo,4,13,FALSE,24
20 BBY,The Clone Wars - A Friend in Need,4,14,FALSE,24
20 BBY,The Clone Wars - Deception,4,15,FALSE,24
20 BBY,The Clone Wars - Friends and Enemies,4,16,FALSE,24
20 BBY,The Clone Wars - The Box,4,17,FALSE,24
20 BBY,The Clone Wars - Crisis on Naboo,4,18,FALSE,24
20 BBY,The Clone Wars - Massacre,4,19,FALSE,24
20 BBY,Tales of the Empire - The Path of Fear,1,1,FALSE,20
20 BBY,The Clone Wars - Bounty,4,20,FALSE,24
20 BBY,The Clone Wars - Brothers,4,21,FALSE,24
20 BBY,The Clone Wars - Revenge,4,22,FALSE,24
20 BBY,The Clone Wars - A War on Two Fronts,5,2,FALSE,24
20 BBY,The Clone Wars - Front Runners,5,3,FALSE,24
20 BBY,The Clone Wars - The Soft War,5,4,FALSE,24
20 BBY,The Clone Wars - Tipping Points,5,5,FALSE,24
20 BBY,The Clone Wars - The Gathering,5,6,FALSE,24
20 BBY,The Clone Wars - A Test of Strength,5,7,FALSE,24
20 BBY,The Clone Wars - Bound for Rescue,5,8,FALSE,24
20 BBY,The Clone Wars - A Necessary Bond,5,9,FALSE,24
20 BBY,The Clone Wars - Secret Weapons,5,10,FALSE,24
20 BBY,The Clone Wars - A Sunny Day in the Void,5,11,FALSE,24
20 BBY,The Clone Wars - Missing in Action,5,12,FALSE,24
20 BBY,The Clone Wars - Point of No Return,5,13,FALSE,24
19 BBY,The Clone Wars - Revival,5,1,FALSE,24
19 BBY,The Clone Wars - Eminence,5,14,FALSE,24
19 BBY,The Clone Wars - Shades of Reason,5,15,FALSE,24
19 BBY,The Clone Wars - The Lawless,5,16,FALSE,24
19 BBY,The Clone Wars - Sabotage,5,17,FALSE,24
19 BBY,The Clone Wars - The Jedi Who Knew Too Much,5,18,FALSE,24
19 BBY,The Clone Wars - To Catch a Jedi,5,19,FALSE,24
19 BBY,The Clone Wars - The Wrong Jedi,5,20,FALSE,24
19 BBY,The Clone Wars - The Unknown,6,1,FALSE,24
19 BBY,The Clone Wars - Conspiracy,6,2,FALSE,24
19 BBY,The Clone Wars - Fugitive,6,3,FALSE,24
19 BBY,The Clone Wars - Orders,6,4,FALSE,24
19 BBY,The Clone Wars - An Old Friend,6,5,FALSE,24
19 BBY,The Clone Wars - The Rise of Clovis,6,6,FALSE,24
19 BBY,The Clone Wars - Crisis at the Heart,6,7,FALSE,24
19 BBY,The Clone Wars - The Disappeared Part I,6,8,FALSE,24
19 BBY,The Clone Wars - The Disappeared Part II,6,9,FALSE,24
19 BBY,The Clone Wars - The Lost One,6,10,FALSE,24
19 BBY,The Clone Wars - Voices,6,11,FALSE,24
19 BBY,The Clone Wars - Destiny,6,12,FALSE,24
19 BBY,The Clone Wars - Sacrifice,6,13,FALSE,24
19 BBY,The Clone Wars - Gone with a Trace,7,5,FALSE,24
19 BBY,The Clone Wars - Deal No Deal,7,6,FALSE,24
19 BBY,The Clone Wars - Dangerous Debt,7,7,FALSE,24
19 BBY,The Clone Wars - Together Again,7,8,FALSE,24
19 BBY,The Clone Wars - The Bad Batch,7,1,FALSE,30
19 BBY,The Clone Wars - A Distant Echo,7,2,FALSE,24
19 BBY,The Clone Wars - On the Wings of Keeradaks,7,3,FALSE,24
19 BBY,The Clone Wars - Unfinished Business,7,4,FALSE,24
19 BBY,The Clone Wars - Old Friends Not Forgotten,7,9,FALSE,24
19 BBY,Episode III Revenge of the Sith,,,FALSE,140
19 BBY,The Clone Wars - The Phantom Apprentice,7,10,FALSE,24
19 BBY,The Clone Wars - Shattered,7,11,FALSE,24
19 BBY,The Bad Batch - Aftermath,1,1,FALSE,30
19 BBY,The Clone Wars - Victory and Death,7,12,FALSE,24
,Tales of the Empire - Devoted,1,2,FALSE,20
,Tales of the Jedi - Resolve,1,6,FALSE,20
19 BBY,The Bad Batch - Cut and Run,1,2,FALSE,30
19 BBY,The Bad Batch - Replacements,1,3,FALSE,30
19 BBY,The Bad Batch - Cornered,1,4,FALSE,30
19 BBY,The Bad Batch - Rampage,1,5,FALSE,30
19 BBY,The Bad Batch - Decommissioned,1,6,FALSE,30
19 BBY,The Bad Batch - Battle Scars,1,7,FALSE,30
19 BBY,The Bad Batch - Reunion,1,8,FALSE,30
19 BBY,The Bad Batch - Bounty Lost,1,9,FALSE,30
19 BBY,The Bad Batch - Common Ground,1,10,FALSE,30
19 BBY,The Bad Batch - Devil's Deal,1,11,FALSE,30
19 BBY,The Bad Batch - Rescue on Ryloth,1,12,FALSE,30
19 BBY,The Bad Batch - Infested,1,13,FALSE,30
19 BBY,The Bad Batch - War-Mantle,1,14,FALSE,30
19 BBY,The Bad Batch - Return to Kamino,1,15,FALSE,30
19 BBY,The Bad Batch - Kamino Lost,1,16,FALSE,30
19–18 BBY,The Bad Batch - Spoils of War,2,1,FALSE,30
19–18 BBY,The Bad Batch - Ruins of War,2,2,FALSE,30
19–18 BBY,The Bad Batch - The Solitary Clone,2,3,FALSE,30
19–18 BBY,The Bad Batch - Faster,2,4,FALSE,30
19–18 BBY,The Bad Batch - Entombed,2,5,FALSE,30
19–18 BBY,The Bad Batch - Tribe,2,6,FALSE,30
18 BBY,The Bad Batch - The Clone Conspiracy,2,7,FALSE,30
18 BBY,The Bad Batch - Truth and Consequences,2,8,FALSE,30
,Tales of the Underworld - A Way Forward,1,4,FALSE,20
,Tales of the Underworld - Friends,1,5,FALSE,20
,Tales of the Underworld - One Warrior to Another,1,6,FALSE,20
18 BBY,The Bad Batch - The Crossing,2,9,FALSE,30
18 BBY,The Bad Batch - Retrieval,2,10,FALSE,30
18 BBY,The Bad Batch - Metamorphosis,2,11,FALSE,30
18 BBY,The Bad Batch - The Outpost,2,12,FALSE,30
18 BBY,The Bad Batch - Pabu,2,13,FALSE,30
18 BBY,The Bad Batch - Tipping Point,2,14,FALSE,30
18 BBY,The Bad Batch - The Summit,2,15,FALSE,30
18 BBY,The Bad Batch - Plan 99,2,16,FALSE,30
19 BBY,The Bad Batch - Confined,3,1,FALSE,30
20 BBY,The Bad Batch - Paths Unknown,3,2,FALSE,30
21 BBY,The Bad Batch - Shadows of Tantiss,3,3,FALSE,30
22 BBY,The Bad Batch - A Different Approach,3,4,FALSE,30
23 BBY,The Bad Batch - The Return,3,5,FALSE,30
24 BBY,The Bad Batch - Infiltration,3,6,FALSE,30
25 BBY,The Bad Batch - Extraction,3,7,FALSE,30
26 BBY,The Bad Batch - Bad Territory,3,8,FALSE,30
27 BBY,The Bad Batch - The Harbinger,3,9,FALSE,30
28 BBY,The Bad Batch - Identity Crisis,3,10,FALSE,30
29 BBY,The Bad Batch - Point of No Return,3,11,FALSE,30
30 BBY,The Bad Batch - Juggernaut,3,12,FALSE,30
31 BBY,The Bad Batch - Into the Breach,3,13,FALSE,30
32 BBY,The Bad Batch - Flash Strike,3,14,FALSE,30
33 BBY,The Bad Batch - The Cavalry Has Arrived,3,15,FALSE,30
34 BBY,Tales of the Empire - Realization,1,3,FALSE,20
10 BBY,Solo A Story,,,FALSE,135
c. 9 BBY–c. 2 BBY,Tales of the Empire - The Path of Anger,1,4,FALSE,20
9 BBY,Obi-Wan Kenobi - Part I,1,1,FALSE,47
9 BBY,Obi-Wan Kenobi - Part II,1,2,FALSE,47
9 BBY,Obi-Wan Kenobi - Part III,1,3,FALSE,47
9 BBY,Obi-Wan Kenobi - Part IV,1,4,FALSE,47
9 BBY,Obi-Wan Kenobi - Part V,1,5,FALSE,47
9 BBY,Obi-Wan Kenobi - Part VI,1,6,FALSE,47
5 BBY,Andor - Kassa,1,1,FALSE,50
5 BBY,Andor - That Would Be Me,1,2,FALSE,50
5 BBY,Andor - Reckoning,1,3,FALSE,50
5 BBY,Andor - Aldhani,1,4,FALSE,50
5 BBY,Andor - The Axe Forgets,1,5,FALSE,50
5 BBY,Andor - The Eye,1,6,FALSE,50
5 BBY,Andor - Announcement,1,7,FALSE,50
5 BBY,Andor - Narkina 5,1,8,FALSE,50
5 BBY,Andor - Nobody's Listening!,1,9,FALSE,50
5 BBY,Andor - One Way Out,1,10,FALSE,50
5 BBY,Andor - Daughter of Ferrix,1,11,FALSE,50
5 BBY,Andor - Rix Road,1,12,FALSE,50
5 BBY,Rebels - Spark of Rebellion,1,1,FALSE,44
5 BBY,Rebels - Droids in Distress,1,3,FALSE,22
5 BBY,Rebels - Fighter Flight,1,4,FALSE,22
5 BBY,Rebels - Rise of the Old Masters,1,5,FALSE,22
5 BBY,Rebels - Breaking Ranks,1,6,FALSE,22
5 BBY,Rebels - Out of Darkness,1,7,FALSE,22
4 BBY,Rebels - Empire Day,1,8,FALSE,22
4 BBY,Rebels - Gathering Forces,1,9,FALSE,22
4 BBY,Rebels - Path of the Jedi,1,10,FALSE,22
4 BBY,Rebels - Idiot's Array,1,11,FALSE,22
4 BBY,Rebels - Vision of Hope,1,12,FALSE,22
4 BBY,Rebels - Call to Action,1,13,FALSE,22
4 BBY,Rebels - Rebel Resolve,1,14,FALSE,22
4 BBY,Rebels - Fire Across the Galaxy,1,15,FALSE,22
4 BBY,Andor - One Year Later,2,1,FALSE,50
4 BBY,Andor - Sagrona Teema,2,2,FALSE,50
4 BBY,Andor - Harvest,2,3,FALSE,50
4 BBY,Rebels - The Siege of Lothal,2,1,FALSE,44
4 BBY,Rebels - The Lost Commanders,2,3,FALSE,22
4 BBY,Rebels - Relics of the Old Republic,2,4,FALSE,22
4 BBY,Rebels - Always Two There Are,2,5,FALSE,22
4 BBY,Rebels - Brothers of the Broken Horn,2,6,FALSE,22
4 BBY,Rebels - Wings of the Master,2,7,FALSE,22
4 BBY,Rebels - Blood Sisters,2,8,FALSE,22
4 BBY,Rebels - Stealth Strike,2,9,FALSE,22
3 BBY,Rebels - The Future of the Force,2,10,FALSE,22
3 BBY,Andor - Ever Been to Ghorman?,2,4,FALSE,50
3 BBY,Andor - I Have Friends Everywhere,2,5,FALSE,50
3 BBY,Andor - What a Festive Evening,2,6,FALSE,50
3 BBY,Rebels - Legacy,2,11,FALSE,22
3 BBY,Rebels - A Princess on Lothal,2,12,FALSE,22
3 BBY,Rebels - The Protector of Concord Dawn,2,13,FALSE,22
3 BBY,Rebels - Legends of the Lasat,2,14,FALSE,22
3 BBY,Rebels - The Call,2,15,FALSE,22
3 BBY,Rebels - Homecoming,2,16,FALSE,22
3 BBY,Rebels - The Honorable Ones,2,17,FALSE,22
3 BBY,Rebels - Shroud of Darkness,2,18,FALSE,22
3 BBY,Rebels - The Forgotten Droid,2,19,FALSE,22
3 BBY,Rebels - The Mystery of Chopper Base,2,20,FALSE,22
3 BBY,Rebels - Twilight of the Apprentice,2,21,FALSE,44
2 BBY,Rebels - Steps Into Shadow,3,1,FALSE,44
2 BBY,Rebels - The Holocrons of Fate,3,3,FALSE,22
2 BBY,Rebels - The Antilles Extraction,3,4,FALSE,22
2 BBY,Rebels - Hera's Heroes,3,5,FALSE,22
2 BBY,Rebels - The Last Battle,3,6,FALSE,22
2 BBY,Rebels - Imperial Supercommandos,3,7,FALSE,22
2 BBY,Rebels - Iron Squadron,3,8,FALSE,22
2 BBY,Rebels - The Wynkahthu Job,3,9,FALSE,22
2 BBY,Rebels - An Inside Man,3,10,FALSE,22
2 BBY,Rebels - Visions and Voices,3,11,FALSE,22
2 BBY,Rebels - Ghosts of Geonosis,3,12,FALSE,44
2 BBY,Rebels - Warhead,3,14,FALSE,22
2 BBY,Rebels - Trials of the Darksaber,3,15,FALSE,22
2 BBY,Rebels - Legacy of Mandalore,3,16,FALSE,22
2 BBY,Andor - Messenger,2,7,FALSE,50
2 BBY,Andor - Who Are You?,2,8,FALSE,50
2 BBY,Rebels - Through Imperial Eyes,3,17,FALSE,22
2 BBY,Andor - Welcome to the Rebellion,2,9,FALSE,50
2 BBY,Rebels - Secret Cargo,3,18,FALSE,22
2 BBY,Rebels - Double Agent Droid,3,19,FALSE,22
2 BBY,Rebels - Twin Suns,3,20,FALSE,22
2 BBY,Rebels - Zero Hour,3,21,FALSE,44
1 BBY,Rebels - Heroes of Mandalore,4,1,FALSE,22
1 BBY,Rebels - In the Name of the Rebellion,4,3,FALSE,44
1 BBY,Rebels - The Occupation,4,5,FALSE,22
1 BBY,Rebels - Flight of the Defender,4,6,FALSE,22
1 BBY,Rebels - Kindred,4,7,FALSE,22
1 BBY,Rebels - Crawler Commandeers,4,8,FALSE,22
1 BBY,Rebels - Rebel Assault,4,9,FALSE,22
1 BBY,Rebels - Jedi Night,4,10,FALSE,22
1 BBY,Rebels - DUME,4,11,FALSE,22
1 BBY,Rebels - Wolves and a Door,4,12,FALSE,22
1 BBY,Rebels - A World Between Worlds,4,13,FALSE,22
1 BBY,Rebels - A Fool's Hope,4,14,FALSE,22
1 BBY,Rebels - Family Reunion and Farewell,4,15,FALSE,22
1 BBY,Tales of the Empire - The Way Out,1,5,FALSE,20
1 BBY,Andor - Make It Stop,2,10,FALSE,50
1 BBY,Andor - Who Else Knows?,2,11,FALSE,50
1 BBY,Andor - Jedha Kyber Erso,2,12,FALSE,50
1 BBY,Rogue One A Story,,,FALSE,133
1 BBY,Episode IV A New Hope,,,FALSE,121
3 ABY,Episode V The Empire Strikes Back,,,FALSE,124
4 ABY,Episode VI Return of the Jedi,,,FALSE,131
9 ABY,The Mandalorian - Chapter 1 The Mandalorian,1,1,FALSE,40
9 ABY,The Mandalorian - Chapter 2 The Child,1,2,FALSE,40
9 ABY,The Mandalorian - Chapter 3 The Sin,1,3,FALSE,40
9 ABY,The Mandalorian - Chapter 4 Sanctuary,1,4,FALSE,40
9 ABY,The Mandalorian - Chapter 5 The Gunslinger,1,5,FALSE,40
9 ABY,The Mandalorian - Chapter 6 The Prisoner,1,6,FALSE,40
9 ABY,The Mandalorian - Chapter 7 The Reckoning,1,7,FALSE,40
9 ABY,The Mandalorian - Chapter 8 Redemption,1,8,FALSE,40
9 ABY,The Mandalorian - Chapter 9 The Marshal,2,1,FALSE,40
9 ABY,The Mandalorian - Chapter 10 The Passenger,2,2,FALSE,40
,Tales of the Empire - The Path of Hate,1,6,FALSE,20
9 ABY,The Mandalorian - Chapter 11 The Heiress,2,3,FALSE,40
9 ABY,The Mandalorian - Chapter 12 The Siege,2,4,FALSE,40
9 ABY,The Mandalorian - Chapter 13 The Jedi,2,5,FALSE,40
9 ABY,The Mandalorian - Chapter 14 The Tragedy,2,6,FALSE,40
9 ABY,The Mandalorian - Chapter 15 The Believer,2,7,FALSE,40
9 ABY,The Mandalorian - Chapter 16 The Rescue,2,8,FALSE,40
9 ABY,The Book of Boba Fett - Chapter 1 Stranger in a Strange Land,1,1,FALSE,55
9 ABY,The Book of Boba Fett - Chapter 2 The Tribes of Tatooine,1,2,FALSE,55
9 ABY,The Book of Boba Fett - Chapter 3 The Streets of Mos Espa,1,3,FALSE,55
9 ABY,The Book of Boba Fett - Chapter 4 The Gathering Storm,1,4,FALSE,55
c. 9 ABY,The Book of Boba Fett - Chapter 5 Return of the Mandalorian,1,5,FALSE,55
c. 9 ABY,The Book of Boba Fett - Chapter 6 From the Desert Comes a Stranger,1,6,FALSE,55
c. 9 ABY,The Book of Boba Fett - Chapter 7 In the Name of Honor,1,7,FALSE,55
9 ABY,The Mandalorian - Chapter 17 The Apostate,3,1,FALSE,40
9 ABY,The Mandalorian - Chapter 18 The Mines of Mandalore,3,2,FALSE,40
9 ABY,The Mandalorian - Chapter 19 The Convert,3,3,FALSE,40
9 ABY,The Mandalorian - Chapter 20 The Foundling,3,4,FALSE,40
9 ABY,The Mandalorian - Chapter 21 The Pirate,3,5,FALSE,40
9 ABY,The Mandalorian - Chapter 22 Guns for Hire,3,6,FALSE,40
9 ABY,The Mandalorian - Chapter 23 The Spies,3,7,FALSE,40
9 ABY,The Mandalorian - Chapter 24 The Return,3,8,FALSE,40
c. 9 ABY,Ahsoka - Part One Master and Apprentice,1,1,FALSE,47
c. 9 ABY,Ahsoka - Part Two Toil and Trouble,1,2,FALSE,47
c. 9 ABY,Ahsoka - Part Three Time to Fly,1,3,FALSE,47
c. 9 ABY,Ahsoka - Part Four Fallen Jedi,1,4,FALSE,47
c. 9 ABY,Ahsoka - Part Five Shadow Warrior,1,5,FALSE,47
c. 9 ABY,Ahsoka - Part Six Far Far Away,1,6,FALSE,47
c. 9 ABY,Ahsoka - Part Seven Dreams and Madness,1,7,FALSE,47
c. 9 ABY,Ahsoka - Part Eight The Jedi the Witch and the Warlord,1,8,FALSE,47
c. 9 ABY,Skeleton Crew - This Could Be a Real Adventure,1,1,FALSE,42
c. 9 ABY,Skeleton Crew - Way Way Out Past the Barrier,1,2,FALSE,42
c. 9 ABY,Skeleton Crew - Very Interesting As an Astrogation Problem,1,3,FALSE,42
c. 9 ABY,Skeleton Crew - Can't Say I Remember No At Attin,1,4,FALSE,42
c. 9 ABY,Skeleton Crew - You Have a Lot to Learn About Pirates,1,5,FALSE,42
c. 9 ABY,Skeleton Crew - Zero Friends Again,1,6,FALSE,42
c. 9 ABY,Skeleton Crew - We're Gonna Be In So Much Trouble,1,7,FALSE,42
c. 9 ABY,Skeleton Crew - The Real Good Guys,1,8,FALSE,42
c. 9 ABY,The Mandalorian and Grogu,,,FALSE,132
34 ABY,Episode VII The Force Awakens,,,FALSE,138
34 ABY,Episode VIII The Last Jedi,,,FALSE,158
35 ABY,Episode IX The Rise of Skywalker,,,FALSE,141

</textarea>
<script>
const KEY="sw_done_map_v2",DAY_KEY="sw_day_reset_timestamp",DAY_HISTORY_KEY="sw_day_history",PREV_MIN_KEY="sw_prev_minutes";
let doneMap=JSON.parse(localStorage.getItem(KEY)||"{}"),
    dayReset=Number(localStorage.getItem(DAY_KEY)||Date.now()),
    dayHistory=JSON.parse(localStorage.getItem(DAY_HISTORY_KEY)||"[]"),
    previousMinutes=Number(localStorage.getItem(PREV_MIN_KEY)||0),
    lastPerDayTarget=0;

const CLEAN_CSV=document.getElementById("csvData").value.trim();

function parseCSV(c){
  if(!c)return[];
  const l=c.split(/\r?\n/),r=[];
  for(let i=1;i<l.length;i++){
    const ln=l[i].trim();
    if(!ln)continue;
    const x=ln.split(",");
    while(x.length<6)x.push("");
    r.push({Year:x[0].trim(),NAME:x[1].trim(),S:x[2].trim(),E:x[3].trim(),Duration:Number(x[5].trim()||"0")});
  }
  return r;
}
const rows=parseCSV(CLEAN_CSV);

function makeKey(r){return`${r.Year}|${r.NAME}|${r.S}|${r.E}`}
function saveDone(){localStorage.setItem(KEY,JSON.stringify(doneMap))}
function formatMinutes(m){const d=Math.floor(m/1440);m%=1440;const h=Math.floor(m/60),mm=m%60;return`${d}d ${h}h ${mm}m`}
function formatHM(m){const h=Math.floor(m/60),mm=m%60;return`${h}h ${mm}m`}

function resetDayStats(){
  let t=0;
  rows.forEach(r=>{const k=makeKey(r);if(doneMap[k])t+=r.Duration});
  const w=Math.max(0,t-previousMinutes);
  dayHistory.push({date:new Date().toISOString().split("T")[0],minutes:w});
  localStorage.setItem(DAY_HISTORY_KEY,JSON.stringify(dayHistory));
  previousMinutes=t;
  localStorage.setItem(PREV_MIN_KEY,previousMinutes);
  dayReset=Date.now();
  localStorage.setItem(DAY_KEY,dayReset);
  updateDailyGoal(lastPerDayTarget);
  updateProjectedFinish();
  updateCurrentPace();
  updateDaysElapsed();
}
document.getElementById("resetDayBtn").addEventListener("click",resetDayStats);

function getWatchedToday(){
  let t=0;
  rows.forEach(r=>{const k=makeKey(r);if(doneMap[k])t+=r.Duration});
  return Math.max(0,t-previousMinutes);
}

function getFranchiseColor(n){
  n=n.toLowerCase();
  if(n.includes("tales of"))return"#992020";
  if(n.includes("episode")||n.includes("phantom menace")||n.includes("attack of the clones")||n.includes("revenge of the sith")||n.includes("solo")||n.includes("rogue one")||n.includes("a new hope")||n.includes("empire strikes back")||n.includes("return of the jedi")||n.includes("force awakens")||n.includes("last jedi")||n.includes("rise of skywalker"))return"#D12A2A";
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

function render(f=""){
  const c=document.getElementById("cardContainer");
  c.innerHTML="";
  let total=0,done=0,totalMin=0,doneMin=0;

  rows.forEach((r,i)=>{
    const k=makeKey(r),e=doneMap[k],
          m=!f||r.NAME.toLowerCase().includes(f.toLowerCase())||r.Year.toLowerCase().includes(f.toLowerCase());
    if(!m)return;
    total++;totalMin+=r.Duration;
    if(e&&e.done){done++;doneMin+=r.Duration}

    const card=document.createElement("div");
    card.className="entry-card"+(e&&e.done?" done":"");
    const bg=getFranchiseColor(r.NAME),
          light=isLightColor(bg),
          tc=light?"#000":"#fff",
          mc=light?"#222":"#9fb4ff";
    card.style.background=bg;
    card.style.borderColor=bg;

    const isFilm=r.NAME.toLowerCase().includes("episode")||
      r.NAME.toLowerCase().includes("phantom menace")||
      r.NAME.toLowerCase().includes("attack of the clones")||
      r.NAME.toLowerCase().includes("revenge of the sith")||
      r.NAME.toLowerCase().includes("solo")||
      r.NAME.toLowerCase().includes("rogue one")||
      r.NAME.toLowerCase().includes("a new hope")||
      r.NAME.toLowerCase().includes("empire strikes back")||
      r.NAME.toLowerCase().includes("return of the jedi")||
      r.NAME.toLowerCase().includes("force awakens")||
      r.NAME.toLowerCase().includes("last jedi")||
      r.NAME.toLowerCase().includes("rise of skywalker");

    let meta=`<b>${r.Year}</b>`;
    if(!isFilm)meta+=` • S${r.S||"-"}E${r.E||"-"}`;
    meta+=` • <span class="badge">${isFilm?"Film":"Series"}</span>`;

    card.innerHTML=`<div class="entry-title" style="color:${tc};">${r.NAME}</div><div class="entry-meta" style="color:${mc};">${meta}</div>`;

    card.addEventListener("click",()=>{
      if(!e||!e.done){
        for(let j=0;j<=i;j++){
          const kk=makeKey(rows[j]);
          doneMap[kk]={done:!0,ts:Date.now()};
        }
      }else{
        delete doneMap[k];
      }
      saveDone();
      render(f);
      startLiveCountdown();
      updateDailyGoal(lastPerDayTarget);
      updateProjectedFinish();
      updateCurrentPace();
      updateDaysElapsed();
    });

    c.appendChild(card);
  });

  const pct=totalMin===0?0:doneMin/totalMin*100;
  document.getElementById("progressBar").style.width=pct+"%";
  document.getElementById("progressText").textContent=Math.round(pct)+"%";
  document.getElementById("headerStats").textContent=
    `${done}/${total} complete • ${formatMinutes(doneMin)} watched • ${formatMinutes(totalMin-doneMin)} left`;
  document.getElementById("summaryLine").textContent=
    `Entries: ${total} • Done: ${done} • Remaining: ${total-done} • Runtime: ${formatMinutes(totalMin)}`;

  updateProjectedFinish();
  updateCurrentPace();
  updateDaysElapsed();
}

function calculateBingeGoal(){
  const v=document.getElementById("endDateInput").value;
  if(!v)return;
  const end=new Date(v),
        today=new Date();
  today.setHours(0,0,0,0);
  const diff=Math.ceil((end-today)/86400000);
  if(diff<=0)return;

  let rem=0;
  rows.forEach(r=>{const k=makeKey(r);if(!doneMap[k])rem+=r.Duration});
  const per=rem/diff;
  lastPerDayTarget=per;

  document.getElementById("summaryLine").innerHTML=
    `Remaining: ${formatMinutes(rem)} • Days Left: ${diff} • <b>Watch ${formatHM(Math.ceil(per))}/day</b>`;

  updateDailyGoal(per);
  updateProjectedFinish();
  updateCurrentPace();
  updateDaysElapsed();
}

let countdownInterval=null;
function startLiveCountdown(){
  const v=document.getElementById("endDateInput").value;
  if(!v)return;
  const end=new Date(v);
  if(countdownInterval)clearInterval(countdownInterval);

  countdownInterval=setInterval(()=>{
    const now=new Date,diff=end-now;
    if(diff<=0){
      document.getElementById("liveCountdown").textContent="Time's up!";
      clearInterval(countdownInterval);
      return;
    }
    const d=Math.floor(diff/86400000),
          h=Math.floor(diff/3600000%24),
          m=Math.floor(diff/60000%60),
          s=Math.floor(diff/1000%60);

    let rem=0;
    rows.forEach(r=>{const k=makeKey(r);if(!doneMap[k])rem+=r.Duration});
    const per=rem/Math.max(d,1);
    lastPerDayTarget=per;

    document.getElementById("liveCountdown").innerHTML=
      `⏳ ${d}d ${h}h ${m}m ${s}s left • 🎬 Need <b>${formatHM(Math.ceil(per))}</b>/day`;

    updateDailyGoal(per);
    updateProjectedFinish();
    updateCurrentPace();
    updateDaysElapsed();
  },1000);
}

function updateDailyGoal(per){
  if(!per||per<=0)return;
  const w=getWatchedToday(),
        pct=Math.min(w/per*100,100),
        fill=document.getElementById("dailyGoalFill");
  fill.style.width=pct+"%";
  let status="";
  if(w>=per){status="🔥 Ahead of schedule";fill.style.background="#4caf50";}
  else if(w>=per*.75){status="⚡ On pace";fill.style.background="#ffc107";}
  else{status="⏳ Behind schedule";fill.style.background="#f44336";}
  document.getElementById("dailyGoalStatus").innerHTML=
    `${status} • ${formatHM(w)} / ${formatHM(Math.ceil(per))}`;
}

function updateProjectedFinish(){
  const startVal=document.getElementById("startDateInput").value;
  if(!startVal){
    document.getElementById("projectedFinish").textContent="";
    return;
  }
  const start=new Date(startVal),
        today=new Date();
  today.setHours(0,0,0,0);
  const daysPassed=Math.max(1,Math.floor((today-start)/86400000));

  let watched=0;
  rows.forEach(r=>{const k=makeKey(r);if(doneMap[k])watched+=r.Duration});
  const avg=watched/daysPassed;
  if(avg<=0){
    document.getElementById("projectedFinish").textContent="";
    return;
  }

  let remaining=0;
  rows.forEach(r=>{const k=makeKey(r);if(!doneMap[k])remaining+=r.Duration});
  const daysNeeded=Math.ceil(remaining/avg),
        finish=new Date();
  finish.setDate(finish.getDate()+daysNeeded);

  document.getElementById("projectedFinish").textContent=
    `Projected Finish: ${finish.toISOString().split("T")[0]}`;
}

function updateCurrentPace(){
  const startVal=document.getElementById("startDateInput").value;
  if(!startVal){
    document.getElementById("currentPace").textContent="";
    return;
  }

  const start=new Date(startVal);
  const today=new Date();
  today.setHours(0,0,0,0);

  const daysPassed=Math.max(1,Math.floor((today-start)/86400000));

  let watched=0;
  rows.forEach(r=>{
    const k=makeKey(r);
    if(doneMap[k])watched+=r.Duration;
  });

  const avg=watched/daysPassed;

  document.getElementById("currentPace").textContent=
    `Current Pace: ${formatHM(Math.ceil(avg))}/day`;
}

function updateDaysElapsed(){
  const startVal=document.getElementById("startDateInput").value;
  if(!startVal){
    document.getElementById("daysElapsed").textContent="";
    return;
  }

  const start=new Date(startVal);
  const today=new Date();
  today.setHours(0,0,0,0);

  const daysPassed=Math.max(1,Math.floor((today-start)/86400000));

  document.getElementById("daysElapsed").textContent=
    `Days Elapsed: ${daysPassed}`;
}

document.getElementById("searchInput").addEventListener("input",e=>{render(e.target.value)});
document.getElementById("clearDoneBtn").addEventListener("click",()=>{
  doneMap={};
  dayHistory=[];
  previousMinutes=0;
  dayReset=Date.now();
  localStorage.setItem(KEY,"{}");
  localStorage.setItem(DAY_HISTORY_KEY,"[]");
  localStorage.setItem(PREV_MIN_KEY,"0");
  localStorage.setItem(DAY_KEY,dayReset);
  render("");
  updateDailyGoal(lastPerDayTarget);
  updateProjectedFinish();
  updateCurrentPace();
  updateDaysElapsed();
});
document.getElementById("calcBtn").addEventListener("click",()=>{
  calculateBingeGoal();
  startLiveCountdown();
});

render();
updateProjectedFinish();
updateCurrentPace();
updateDaysElapsed();
</script>
</body>
</html>

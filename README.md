# Gamesforlif7672.github.io
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Dice If You Dare</title>
<style>
    body { font-family: Arial, sans-serif; margin: 40px; }
    h1 { font-size: 24px; }
    h2 { font-size: 20px; }
    button { margin: 5px; padding: 10px 20px; font-size: 16px; }
    .dice-row { display: flex; flex-wrap: wrap; margin-bottom: 20px; }
    .dice-row button { flex: 1 1 100px; }
    #menu, #upgrades { display: none; }
</style>
</head>
<body>

<h1>Welcome to Dice If You Dare!</h1>
<div id="pointsDisplay">Points: 0</div>

<div id="diceMenu" class="dice-row"></div>

<button id="menuButton">Menu</button>

<div id="menu">
    <h2>Menu</h2>
    <div id="menuPoints"></div>
    <button data-menu="dice">Dice</button>
    <button data-menu="upgrades">Upgrades</button>
    <button data-menu="draw">Draw</button>
    <button data-menu="rebirth">Rebirth</button>
    <button data-menu="prestige">Prestige</button>
    <button id="closeMenu">Close</button>
</div>

<div id="upgrades">
    <h2>Upgrades</h2>
    <div id="upgradePoints"></div>
    <div id="upgradeButtons"></div>
    <button id="backFromUpgrades">Back</button>
</div>

<div id="result"></div>

<script>
let points = 0;
let diceNames = ["D6","D8","D10","D12","D20","D100"];
let diceMax = [6,8,10,12,20,100];
let diceUpgrades = [1,1,1,1,1,1];
let diceUnlocked = [true,false,false,false,false,false];
let unlockCosts = [0,2500,12500,50000,250000,500000];
let rollCosts = [0,250,1250,5000,25000,50000];
let baseMultipliers = [1,1,1,1,1,1];

function calculateMultipliers() {
    for (let i = 0; i < diceMax.length; i++) {
        let avgRoll = Math.ceil((diceMax[i]+1)/2);
        baseMultipliers[i] = Math.ceil(rollCosts[i]/avgRoll);
        if (baseMultipliers[i]<1) baseMultipliers[i]=1;
    }
}
calculateMultipliers();

function updatePoints() {
    document.getElementById("pointsDisplay").innerText = "Points: " + points;
    document.getElementById("menuPoints").innerText = "Points: " + points;
    document.getElementById("upgradePoints").innerText = "Points: " + points;
}

function createDiceButtons() {
    let diceMenu = document.getElementById("diceMenu");
    diceMenu.innerHTML = "";
    diceNames.forEach((name,i)=>{
        let btn = document.createElement("button");
        if(!diceUnlocked[i]){
            btn.innerText = `${name} (Unlock: ${unlockCosts[i]})`;
        } else {
            btn.innerText = `${name} (Roll: ${rollCosts[i]})`;
        }
        btn.onclick = () => {
            if(!diceUnlocked[i]){
                if(points>=unlockCosts[i]){
                    points -= unlockCosts[i];
                    diceUnlocked[i]=true;
                    document.getElementById("result").innerText = `${name} unlocked!`;
                    updatePoints();
                    createDiceButtons();
                } else {
                    document.getElementById("result").innerText = `Not enough points to unlock ${name}`;
                }
                return;
            }
            if(points>=rollCosts[i]){
                points-=rollCosts[i];
                let roll = Math.floor(Math.random()*diceMax[i])+1;
                let multiplier = baseMultipliers[i]*diceUpgrades[i];
                let total = roll*multiplier;
                points += total;
                document.getElementById("result").innerText = `Rolled ${name}: ${roll} × ${multiplier} = ${total}`;
                updatePoints();
            } else {
                document.getElementById("result").innerText = `Not enough points to roll ${name}`;
            }
        };
        diceMenu.appendChild(btn);
    });
}
createDiceButtons();

// Menu logic
document.getElementById("menuButton").onclick = ()=>{
    document.getElementById("menu").style.display = "block";
    document.getElementById("diceMenu").style.display = "none";
};

document.getElementById("closeMenu").onclick = ()=>{
    document.getElementById("menu").style.display = "none";
    document.getElementById("diceMenu").style.display = "flex";
};

document.querySelectorAll("#menu button[data-menu]").forEach(btn=>{
    btn.onclick = ()=>{
        let action = btn.getAttribute("data-menu");
        if(action==="dice"){
            document.getElementById("menu").style.display="none";
            document.getElementById("diceMenu").style.display="flex";
        } else if(action==="upgrades"){
            showUpgrades();
        } else {
            document.getElementById("result").innerText = `${action} menu coming soon!`;
        }
    };
});

// Upgrades menu
function showUpgrades(){
    document.getElementById("menu").style.display="none";
    document.getElementById("diceMenu").style.display="none";
    document.getElementById("upgrades").style.display="block";
    let upgradeDiv = document.getElementById("upgradeButtons");
    upgradeDiv.innerHTML="";
    diceNames.forEach((name,i)=>{
        let btn = document.createElement("button");
        let cost = Math.floor((i===0?50:unlockCosts[i])*Math.pow(1.15+i*0.1,diceUpgrades[i]-1));
        btn.innerText = `${name} Upgrade (Level: ${diceUpgrades[i]}, Cost: ${cost})`;
        btn.onclick = ()=>{
            if(!diceUnlocked[i]){
                document.getElementById("result").innerText = `${name} not unlocked yet!`;
                return;
            }
            if(points>=cost){
                points-=cost;
                diceUpgrades[i]++;
                document.getElementById("result").innerText = `${name} upgraded! Level: ${diceUpgrades[i]}`;
                updatePoints();
                showUpgrades();
            } else {
                document.getElementById("result").innerText = `Not enough points! Upgrade cost: ${cost}`;
            }
        };
        upgradeDiv.appendChild(btn);
    });
}

document.getElementById("backFromUpgrades").onclick = ()=>{
    document.getElementById("upgrades").style.display="none";
    document.getElementById("diceMenu").style.display="flex";
};

</script>
</body>
</html>

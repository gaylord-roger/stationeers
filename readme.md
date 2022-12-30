```mipsasm
alias robot d0
alias leverToReturn d1
alias mode r0
alias tmpValue r1
alias posX r2
alias posY r3
alias posZ r4
alias mineableAmount r5
alias nextZPos r6
alias batteryCharge r7
s robot Mode RobotMode.None

RestartnextZPosInit:
move nextZPos 15

# Called when telling the robot to go mining
Initialize:
s robot TargetX -8
s robot TargetY -1
s robot TargetZ 20
s robot Mode RobotMode.MoveToTarget
WaitUntilOnPosition:
s db Setting 5
yield
l mode robot Mode
beqz mode Initialized
j WaitUntilOnPosition

Initialized:
s robot Mode RobotMode.Roam

Roaming:
s db Setting 8
jal CheckReturnToBase
yield
l mode robot Mode
beqz mode Initialize
# Check if it's full, if so unload
beq mode RobotMode.StorageFull Return
# Check to make sure it has ore to mine, if not explore
l mineableAmount robot MineablesInQueue
beq mineableAmount 0 ExploreOut
j Roaming

# Called when AIMEe is full and needs to return to drop off ore
Return:
# Cache previous mining position so it can return here after unloading ore
l posX robot PositionX
l posY robot PositionY
l posZ robot PositionZ
# You'll be your own chute input coords here
s robot TargetX -8
s robot TargetY 0
s robot TargetZ 1
s robot Mode RobotMode.MoveToTarget
yield

# This function will keep looping until it returns home
Returning:
s db Setting 9
yield
l mode robot Mode
beqz mode Returned
j Returning

# Start unloading once returned home
Returned:
s robot Mode RobotMode.Unload

# Unload and charge batterie
Unloading:
s db Setting 10
yield
ls tmpValue robot 0 ChargeRatio
blt tmpValue 0.95 Unloading # loop until bat is full
l mode robot Mode
beqz mode Leave
j Unloading

Leave:
jal CheckReturnToBase
s robot TargetX posX
s robot TargetY posY
s robot TargetZ posZ
s robot Mode RobotMode.MoveToTarget

# Keep looping until either it finds mineables or reaches its original position
Leaving:
s db Setting 11
jal CheckReturnToBase
yield
l mineableAmount robot MineablesInVicinity
bge mineableAmount 1 Initialize
l mode robot Mode
beqz mode Initialize
j Leaving

# This function will tell AIMEe to explore out on the x axis in the positive direction until it finds mineables
ExploreOut:
add nextZPos nextZPos 10

Explore:
s robot TargetZ nextZPos
s robot Mode RobotMode.MoveToTarget

ExploreNow:
s db Setting 12
jal CheckReturnToBase
l posY robot PositionY
l posX robot PositionX
s robot TargetY posY
s robot TargetX posX
l mineableAmount robot MineablesInVicinity
bge mineableAmount 1 RestartnextZPosInit
l mode robot Mode
beqz mode ExploreOut
j ExploreNow

CheckReturnToBase:
l tmpValue leverToReturn Open
bgt tmpValue 0 Return
ls tmpValue robot 0 ChargeRatio
blt tmpValue 0.1 Return
j ra
```
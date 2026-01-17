```
PROGRAM PLC_PRG
VAR 
	Reset : BOOL := FALSE;
    tStep      : TON;    // Timer for slow down the movement
	Recog	: INT := 0;
	X : INT := 100;
	Y : INT := 0;
	Shape1 : BOOL := TRUE;
	Shape2 : BOOL := TRUE;
	Robot_x : INT := 0;
	Robot_y : INT := 0;
	
	fbPosA : PosA;
	fbPosB : PosB;
	fbShape1 : Shape1;
	fbShape2 : Shape2;
	fbStartCon : Start;
END_VAR
```
```
// ST - 8 lines
// LD - 8 lines
// FBD - 8 lines
CASE GVL.State OF
	0 : // Wait for START button
		IF GVL.Start THEN
			fbStartCon(Exec := TRUE);
		END_IF
	1 : //Start moving box from IDLE pos to A pos
		IF GVL.Conv_Mov THEN
			tStep(IN := TRUE, PT := T#5MS); // Usage of tim to slow down the anim
			IF tStep.Q THEN
				tStep(IN := FALSE);
				X := X - 1; //Absolute mov - prom 100 to 0 -> pos A
			END_IF
			// Pos A acquired
			IF X <= -150 THEN // 0 -> Pos A
				fbPosA(Exec := TRUE);
				//GVL.Conv_Mov := FALSE;
				//GVL.Conv_Mov := FALSE;
				//GVL.Show_c := FALSE; // Make circe visible -> simulation of another given object into the box
				//GVL.PosA_Sns := TRUE;// LED indicator that box with circ is on Pos A
				//GVL.State := 2;
			END_IF
		END_IF
	2 : //If green btn is pressed continue moving box with circle to pos B
		IF	GVL.Btn_G THEN
			GVL.Conv_Mov := TRUE;
			IF GVL.Conv_Mov THEN
				GVL.PosA_Sns := FALSE; // Switch off LED indicator
				tStep(IN := TRUE, PT := T#5MS); //Bcs of slow down animation
				IF tStep.Q THEN
					tStep(IN := FALSE);
					X := X - 1;
				END_IF
			END_IF
			IF X <= -275 THEN // -275 -> end of horizontal conv
				GVL.State := 3;
			END_IF
		END_IF
	3 : //Continue from end of horizontal conv into bottom of vertical conv
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			Y := Y + 1;
		END_IF
		
		IF Y >= 300 THEN // Bottom of vertical conv achieved
			GVL.State := 4;
		END_IF
	4: // Move into pos B
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			X := X + 1;
		END_IF
		
		IF X >= 5 THEN // Pos B achieved
			fbPosB(Exec := TRUE);
			//GVL.PosB_Sns := TRUE;
			//GVL.State := 5;
			//GVL.Btn_G := FALSE;
		END_IF
	5 : // Move to the end of conv
		IF GVL.Btn_G THEN
			GVL.PosB_Sns := FALSE;
			tStep(IN := TRUE, PT := T#5MS);
			IF tStep.Q THEN
				X := X + 1;
			END_IF
			
			IF X >= 235 THEN // End achieved
				GVL.State := 6;
			END_IF
		END_IF
	6 : // Move to the origin horizontal conv
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			Y := Y - 1;
		END_IF
		
		IF Y <= 0 THEN // Origin conv achieved
			GVL.State := 7;
		END_IF
	7 : // Move under camera
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			X := X - 1;
		END_IF
		
		IF X <= 145 THEN // Camera pos achieved
			GVL.State := 8;
		END_IF
	8 : // Object color recognition by CAM
		IF NOT Shape1 THEN
			fbShape1(Exec := TRUE);
			//GVL.Shape1_cam := TRUE;  // Green
			//GVL.Shape2_cam := FALSE;
			//GVL.State := 9;
		END_IF
		
		IF NOT Shape2 THEN
			fbShape2(Exec := TRUE);
			//GVL.Shape1_cam := FALSE; //Red
			//GVL.Shape2_cam := TRUE;
			//GVL.State := 9;
		END_IF
	9 : // Move to robot arm position
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			X := X - 1;
		END_IF
		
		IF X <= -3 THEN
			GVL.State := 10;
		END_IF
	10: // Grab the object with robot arm
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			Robot_y := Robot_y + 1;
		END_IF
		IF Robot_y >= 155 THEN // Object was grasped
			GVL.State := 11;
		END_IF
	11 : // Move robot arm into IDLE pos
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			Robot_y := Robot_y - 1;
			Y := Y - 1;
		END_IF
		
		IF Robot_y <= 0 THEN // IDLE pos achieved
			GVL.State := 12;
		END_IF
	12 : // Move robot arm into the GREEN storage position
		tStep(IN := TRUE, PT := T#5MS); 
		IF tStep.Q THEN
			tStep(IN := FALSE);
			Robot_x := Robot_x + 1;
			X := X + 1;
		END_IF
		IF NOT Shape1 THEN
			IF Robot_x >= 56 THEN
				GVL.State := 13;
			END_IF
		ELSE
			IF Robot_x >= 146 THEN
				GVL.State := 13;
			END_IF
		END_IF
// 		IF Robot_x >= 56 THEN // Storage for GREEN box achieved    TODO: Storage for RED box
// 			State := 13;
// 		END_IF
	13 : //Move robot arm back to IDLE pos
		tStep(IN := TRUE, PT := T#5MS);
		IF tStep.Q THEN
			tStep(IN := FALSE);
			Robot_x := Robot_x - 1;
		END_IF
		
		IF Robot_x <= 0 THEN // IDLE pos achieved
			GVL.State := 14;
		END_IF
	14 : //Reset
		IF Reset THEN
			GVL.Start := FALSE;
			GVL.Show_c := TRUE;
			GVL.Conv_Mov := FALSE;
			GVL.Btn_G := FALSE;
			
			X := 100;
			Y := 0;
			GVL.PosA_Sns := FALSE;
			GVL.PosB_Sns := FALSE;
			Shape1 := TRUE;
			Shape2 := TRUE;
			
			GVL.Shape1_cam := FALSE;
			GVL.Shape2_cam := FALSE;
			
			Robot_x := 0;
			Robot_y := 0;
			
			GVL.State := 0;
		END_IF
END_CASE
```
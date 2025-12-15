void screen()
{
  char phase1[4];
  char phase2[4];
  char phase3[4]; 
  char angle[4];
  char pot_ch[8];
  
  // Mensagem de offset salvo tem prioridade por ~2 s
  if (showOffsetMsg) {
    if (millis() - offsetMsgTime < 2000) {
      lcd.setCursor(0,0);
      lcd.print("Offset TX");
      lcd.print(lastOffsetTx);
      lcd.print(" salvo   ");
      
      lcd.setCursor(0,1);
      lcd.print("                ");
      return;
    } else {
      showOffsetMsg = false;
    }
  }

  switch(menu)
  {
    case 0:
      lcd.setCursor(0,0);
      lcd.print(" Do'A Beam Team");
      lcd.setCursor(0,1);
      lcd.print("IEEE  AP-S SDC21");
      break;
    
    case 1:
      lcd.setCursor(0,0);
      lcd.print("Phase Shifts:  ");
      lcd.setCursor(0,1);
      sprintf(phase1, "%3d", val_1m);
      sprintf(phase2, "%3d", val_2m);
      sprintf(phase3, "%3d", val_3m);
      lcd.print("  0");
      lcd.print(degree);
      lcd.print(phase1);
      lcd.print(degree);
      lcd.print(phase2);
      lcd.print(degree);
      lcd.print(phase3);
      lcd.print(degree);
      break;
    
    case 2:
      lcd.setCursor(0,0);
      lcd.print("Power Received:");
      lcd.setCursor(0,1);
      sprintf(pot_ch, "%3.1f", pot);
      lcd.print("        ");
      lcd.print(pot_ch);
      lcd.print("dBm");
      break;
    
    case 3:
      lcd.setCursor(0,0);
      lcd.print("Rx Waves Phase: ");
      lcd.setCursor(0,1);
      lcd.print("To be done...   ");
      break;

    case 4:
      lcd.setCursor(0,0);
      lcd.print("Tx Beam Angle:  ");
      lcd.setCursor(0,1);
      sprintf(angle, "%3d", teta);
      lcd.print(angle);
      lcd.print(degree);
      lcd.print("            ");
      break;

    case 5:
      lcd.setCursor(0,0);
      lcd.print("Auto Scan Mode: ");
      lcd.setCursor(0,1);
      if (autoScanActive) {
        sprintf(angle, "%3d", teta);
        lcd.print("ON  ");
        lcd.print(angle);
        lcd.print(degree);
        lcd.print("      ");
      } else {
        lcd.print("OFF            ");
      }
      break;
  }
}

Rychlost miestnosti je nastavena na 30, t. j. jedna sekunda predstavuje 30 krokov.

Snazil som sa vsetok kod pisat v "Create a piece of code", ostatne je pisane v hranatych zatvorkach "[", "]" a vysvetlene.

### rm_Main (creation code):
	// ak nehra hudba v pozadi, spusti ju
	if(!audio_is_playing(snd_Background)) {
    		audio_play_sound(snd_Background, 1, true);
	}
	// nastav vertikalny pohyb pozadia v tejto miestnosti na 2px/krok (t. j. pozadie sa
	// pohybuje smerom nadol)
	background_vspeed[0] = 2;
	// pomocna premenna, oznacujuca koniec hry
	ended = false;

### rm_Game (creation code):
	randomize();
	// vytvorim dve instancie objektu obj_Enemy nad viditelnou castou obrazu
	instance_create(random(200)+60, -200, obj_Enemy);
	instance_create(random(200)+60, -400, obj_Enemy);
	// vertikalny pohyb pozadia
	background_vspeed[0] = 2;

### rm_Tutorial (creation code):
	// zastavi animaciu postav Mayor a Sherif
	with(obj_Mayor) image_speed = 0;
	with(obj_Sherif) image_speed = 0;
	// prepne sa do modu pre prvu cast tutorialu (neda sa strielat, hybat, ...)
	with(obj_Score) tutor = 1;

### rm_HighScore (creation code):
	// nastavim verikalny pohyb pozadia
	background_vspeed[0] = 2;

### rm_test:
	nema creation code

### obj_Bullet:
* Create event:

		// raz prehra zvuk snd_Pistol
		audio_play_sound(snd_Pistol, 10, false);
* Outside Room event:

		// znici instanciu objektu obj_Bullet
		instance_destroy();

### obj_enmBullet
* rovnako ako obj_Bullet (zabezpecuje aby sa nepriatelia nestrielali medzi sebou)

### obj_Sherif:
* Create event:

		// pomocna premenna, obmedzujuca pohyb ci moznost strelby
		tutor = 0;
		// pocitadlo vystrelov a pocitadlo pohybov pre Tutorial
		shotCounter = 0;
		moveCounter = 0;
* Collision with obj_Wall event:

		// zastavi pohyb serifa
		speed = 0;
		// znici instanciu, s ktorou sa zrazil, t. j. stenu vytvorenu pri zaciatku pohybu
		with(other) instance_destroy();
		// ak som v casti tutorialu, kde sa nehybe pozadie, t.j. tutorial pohybu, tak zastavi
		// animaciu
		if (background_vspeed[0] == 0) image_speed = 0;
* Global Left Pressed event:

		// ak sme tukli v hornej casti miestnosti a sme v casti tutorialu, kde sa da
		// strielat
		if(mouse_y < 960 && (tutor == 0 || tutor == 3)) {
			shotCounter++;
			// vytvor instanciu objektu obj_Bullet
			with(instance_create(x, y, obj_Bullet)) {
				// nastav tejto instancii smer a rychlost
				direction = point_direction(x, y, mouse_x, mouse_y);
				speed = 90;
			}
		}
		// ak v dolnej casti obrazovky a sme v casti tutorialu, kde sa da hybat
		else if(mouse_y >= 960 && (tutor == 0 || tutor == 2)) {
			// znic pripadne existujuce instancie pomocneho objektu obj_Wall
			with(obj_Wall) instance_destroy();
			// vytvor instanciu objektu obj_Wall 15px od miesta tuknutia (sirka serifa je
			// priblizne 30px)
    			if(mouse_x > x) temp_x = mouse_x + 15;
    			else temp_x = mouse_x - 15;
			instance_create(temp_x, y, obj_Wall);
			// nastavime smer serifovho pohybu vodorovne k novovytvorenej instancii
			direction = point_direction(x, y, mouse_x, y);
			// nastavim rychlost serifovho pohybu na 7px/krok
			speed = 7;
			// spustim animaciu pohybu, ak nahodou nebola
			image_speed = 1;
			moveCounter++;
		}

### obj_Mayor:
* Create event:

		// nastavim pomocnu premennu tutor na 0 (rozhoduje o zabitelnosti starostu)
		tutor = 0;
* Step event:

		// ked sa animacia dostane na obrazok 33, zastavi sa
		// animacia chodze ma 17 obrazkov, cize pocas tejto animacie sa nezastavi, az ked zmenime
		// Sprite na spr_Death
		if image_index == 33 then image_speed = 0;
* Collision with obj_Bullet event:

		// znici instanciu, s ktorou sa zrazil
		with(obj_Bullet) instance_destroy();
		if tutor==0 {
			// vytvori instanciu objektu obj_End
			instance_create(290, 568, obj_End);
			// zmeni sprite na spr_Death
    			sprite_index = spr_Death;
			// raz prehra zvuk smrti
			audio_play_sound(snd_Dead1, 10, false);
			// zastavi animaciu serifa
			with(obj_Sherif) image_speed = 0;
		}
* Collision with obj_enmBullet event:

		rovnako ako pre obj_Bullet (len namiesto with(obj_Bullet) pouzivam with(obj_enmBullet))

### obj_Enemy:
* Create event:

		randomize();
		// lokalna premenna shot oznacuje, ci dana instancia objektu uz vystrelila
		var shot;
		// shot nastavime na false
		shot = false;
		// premenna shoot_now oznacuje, kedy ma dana instancia vystrelit, nahodne vygenerujeme
		shoot_now = random(200)+500;
		// nastavim instancii vertikalnu rychlost 10px za krok
		vspeed = 10;
* Step event:

		// ak uz ma strielat a este nevystrelil, tak vystreli (raz) objekt obj_enmBullet (aby sa
		// nestrielali medzi sebou)
		if ((y > shoot_now) && !shot) {
			with(instance_create(x, y, obj_enmBullet)) {
				// nastavim smer na obj_Mayor, trochu vyssie, aby ho zasiahli tam, kde potom "krvaca"
        direction = point_direction(x, y, obj_Mayor.x, obj_Mayor.y - 50);
				// nastavim rychlost na 30px za krok
        speed = 30;
			}
			// uz vystrelil
			shot = true;
		}
* Collision with obj_Bullet event:

		// zvysim skore
		score += 10;
		// prehram nahodne vybrany zvuk umierania
		// irandom vygeneruje nahodne cislo medzi 0-4 (ano aj 4)
		n = irandom(4);
		switch(n) {
    			case 0: audio_play_sound(snd_Dead0, 10, false);
        			break;
    			case 1: audio_play_sound(snd_Dead1, 10, false);
        			break;
    			case 2: audio_play_sound(snd_Dead2, 10, false);
        			break;
    			case 3: audio_play_sound(snd_Dead3, 10, false);
        			break;
    			default: audio_play_sound(snd_Dead4, 10, false);
		}
		// znicim instanciu objektu other, co je instancia obj_Bullet, s ktorou kolidujem
		with(other) instance_destroy();
		// zabranim vystrelu
		shot = true;
		// presuniem instanciu mimo miestnost
		y = 1500;
* Outside Room event:

		randomize();
		// shot nastavim na false
		shot = false;
		// znovu vygenerujem poziciu kedy ma vystrelit
		shoot_now = random(200)+500;
		// umiestnim ho do lavej alebo pravej polovice (aby sa nestretol so starostom)
		temp = irandom(1);
		if(temp == 0) x = random(270) + 20;
		else x = random(300) + 440;
		// y nastavim na -200, kedze ma nastaveny vspeed, tak zide dole
		y = -200;

### obj_Score:
* Create event:

		// set score value to 0
		score = 0;
		// counter of bonus enemies
		i = 1;
		// sets score font to f_score
		draw_set_font(f_score);
		// pomocna premenna (ohranicuje pribudanie nepriatelov)
		tutor = 0;
* Step event:

		// pri kazdej stovke zvysim pocet nepriatelov o 1
		if(score == i*100 && tutor == 0) {
			i++;
			instance_create(100, 1500, obj_Enemy);
		}
* Draw event:
		
		// vykresli hodnotu skore na poziciu x = 10, y = 5 s prefixom "Score:"
		[Draw the value of score]

### obj_Star:
* Left Pressed event:

		// pouzival som na presun do testovacej miestnosti (testovanie animacie, zvuku, ...)
		//room_goto(rm_test);

### obj_Play:
* Left Pressed event:

		// presunie hraca do miestnosti rm_Game
		room_goto(rm_Game);

### obj_Highscore:
* Left Pressed event:

		// presunie hraca do miestnosti rm_HighScore
		room_goto(rm_HighScore);

### obj_Tutorial:
* Left Pressed event:

		// presunie hraca do miestnosti tutorialu
		room_goto(rm_Tutorial);

### obj_Quit:
* Left Pressed event:

		// ukonci hru
		game_end();

### obj_Menu:
* Left Pressed event:

		// presunie hraca do hlavnej ponuky
		room_goto(rm_Main);

### obj_Leaderboard:
* Draw event:

		// nastavi font na f_score (rovnaky ako skore)
		draw_set_font(f_score);
		// vykresli tabulku top 10 hracov v obdlzniku vymedzenom dvomi bodmi
		// (lavy horny a pravy dolny roh)
		draw_highscore(x+20, y+15, 695, y+695);

### obj_Reset:
* Left Pressed event:

		// vypyta si heslo na resetovanie tabulky HighScore
		var ans = get_string("What's the password?", "");
		// heslo je "Answer", skontrolujem a vycistim tabulku
		if ans == "Answer" highscore_clear();

### obj_TutorialText:
* Create event:

		// nastavil som rychlost animacie na 0, cize vzdy zobrazuje 1 obrazok z tutorialu
		image_speed=0;
* Global Left Pressed event:

		// kedze pri kazdom kroku chcem robit nieco ine vytvoril som switch, ktory kontroluje
		// kde sme v tutoriali 
		switch(image_index) {
			// ak sme na zaciatku
			case 0:
				// prejdeme na dalsi obrazok
				image_index++;
				// vytvorime instanciu serifa a nastavime mu: 
				with(instance_create(381,1082,obj_Sherif)) {
					// aby nestrielal ani sa nehybal a zastavime animaciu
					tutor=1;
					image_speed=0;
				}
        			break;
			case 1:
        			image_index++;
        			// vytvorime instanciu starostu a nastavime mu:
        			with(instance_create(381,868,obj_Mayor)) {
					// ze sa ho neda zabit a zastavime animaciu
					tutor=1;
					image_speed=0;
				}
				break;
			case 2:
				// len prejdeme ne dalsi obrazok
				image_index++;
				break;
			case 3:
				image_index++;
				// vytvorime instanciu banditu, bez pohybu a bez animacie
        			with(instance_create(344,143,obj_Enemy)) {
					image_speed=0;
					speed=0;
        			}
        			break;
    			case 4:
        			// odstranime banditu
        			with(obj_Enemy) instance_destroy();
        			image_index++;
        			// serifa posunieme do dalsieho stupna tutorialu (moze sa len hybat)
        			with(obj_Sherif) tutor=2;
        			break;
    			case 5:
        			with(obj_Sherif) {
        				// ak sa hrac so serifom uspesne 3-krat pohol, serif sa znovu nemoze hybat a
					// cakame na dalsie tuknutie, ktore posunie tutorial do dalsieho kroku
					if moveCounter == 3 {
						tutor=1;
						moveCounter++;
					}
					else if moveCounter == 4 {
						// ak sme sa daneho tuknutia dockali, pripravime serifa na dalsi stupen
						// tutorialu , teraz moze len strielat
						tutor=3;
						// a posunieme sa dalej
						with(obj_TutorialText) image_index++;
					}
        			}
        			break;
			case 6:
				with(obj_Sherif) {
					// ak hrac 3-krat vystrelil, cakame rovanko ako pri pohybe
					if shotCounter==3 {
						shotCounter++;
						tutor=1;
					}
					else if shotCounter==4 with(obj_TutorialText) image_index++;
				}
				break;
			case 7:
				// na konci spustime animacie serifa, starostu aj pozadia a pridame tlacidlo
				// navratu do hlavneho menu
				image_index++;
      				with(obj_Sherif) {
      					image_speed=1;
				}
      				with(obj_Mayor) {
      					image_speed=1;
				}
      				background_vspeed[0] = 2;
      				instance_create(640, 1184, obj_Menu);
			case 8:
				// teraz sa nachadzame v testovacej zone, kde chodia len 2 banditi
      				// tu si moze hrac vyskusat ako prebieha hra (starostu sa tu neda zabit)
      				with(obj_Sherif) {
					tutor=0;
				}
      				instance_create(200, -300, obj_Enemy);
      				instance_create(500, -500, obj_Enemy);
      				// znicime instanciu objektu tutorial, aby zbytocne nezaberala miesto v pamati
				instance_destroy();
      				break;
		}

### obj_End:
* Create event:

		// znici instancie objektu obj_Enemy
		with(obj_Enemy) instance_destroy();
		// zastavi pohyb pozadia
		background_vspeed[0] = 0;
		// nastavi premennu tutor objektu obj_Sherif na 1 (cize nemoze strielat, ani hybat sa)
		with(obj_Sherif) tutor = 1;
		if score > 0 {
			// vypyta si od hraca meno do tabulky highscore
			var name = get_string("What do you want to be known as?", "");
			// pridava len hracov, ktori zadali meno (vyber do highscore je vramci funkcie implementovanej vo frameworku)
			if(name != '') highscore_add(name, score);
		}
		// vytvori instanciu tlacitka pre prechod do hlavneho menu
		instance_create(640, 1184, obj_Menu);
* Draw event:

		// najprv vykresli seba
		draw_self();

		// vykresli hodnotu skore na poziciu x, y objectu obj_End s prefixom "Score:" (x, y nie su v strede, ani lavy horny roh, nastavenie vramci GameMaker, presna pozicia)
		[Draw the value of score]

### obj_Wall (pomocny, neviditelny objekt na zastavenie pohybu serifa):
* Collision with obj_sherif event:

		// znici instanciu objektu obj_Wall
		instance_destroy();

### obj_test
* neobsahuje ziadny kod, sluzil len na testovanie animacie

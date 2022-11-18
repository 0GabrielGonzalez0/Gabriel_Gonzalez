# Gabriel_Gonzalez
#define TYPING_INTER_KEY_DELAY 20
#define nNOENTER true
#define NOENTER false

#define NOENTER 2
#define SLOW 6
#define HALT 9
#define TYPING 10
#define PHONE
#ifdef PHONE
int phone = true;

char myString[] = "  This is BeeTea Engineer, DHR, making a test call.\n "
                  " Please hang up the B-leg \n "
                  " I am testing  the new Typing UK app  \n "
                  " Hello Doug here  \n "
                  " Can   you see my typing?  \n "
                  " The Quick brown fox jumps over the lazy dog  \n "
                  " I am testing the    new Relay UK app \n "
                  //" 1 2 3 4 5 6 7 8 9 ! Ã‚Â£ $ % ^ & * ( ) _- + = { } [ ] ~ # : ; @ ' < > , . / ?"
                  ;

#else
int phone = false;
/* Agent typing - The \n indicate end of sentence. */
char myString[] = " Welcome  to Chit  Chat UK   \n "
                  " Please  type your  name?   \n  "
                  " You are talking  to Jack  \n "
                  " How May I help you today  \n "
                  " Okay I   will type  back?   \n  "
                  " The Quick  brown  fox jumps  over the  lazy dog  \n "
                  " Hello  Caller the  other party has cleared.  \n "
                  " \n ";

#endif

/*========================================================
   SEC: 3.0 - Variables
  ======================================================*/

int slowSpeed = true;
int waiting = true;

// the following variables are unsigned longs because the time, measured in
// milliseconds, will quickly become a bigger number than can be stored in an int.
unsigned long lastTs = 0;  // the last timestamp

char * ptr;

void setup() {
  Serial1.begin(9600);

  // don't need to set anything up to use DigiKeyboard
  ptr = myString;

  pinMode( NOENTER, INPUT_PULLUP ); // suppress \n <ENTER>
  pinMode( SLOW, INPUT_PULLUP ); // speed
  pinMode( HALT, INPUT_PULLUP );
  pinMode( TYPING, INPUT_PULLUP );  // used for WiredOR
  pinMode( LED_BUILTIN, OUTPUT );

  waiting = true;
  Serial1.println(" hello , Use slide switches to select features\n");
  Serial1.println(" _/ _ use CRLRF\n _/ _ speed\n _/ _ HALT\n _/ _ typing\n");

}

void polledTimeSlice() {
  unsigned long now = millis();  // the last timestamp

  if ( now - lastTs > 300 ) {
    lastTs = now;
    
    if ( digitalRead( TYPING ) ) {
      waiting = false;
    }
  }
}

void LED( int on ) {

  if ( on ) {
    // ASSERT LOW
    pinMode( LED_BUILTIN, OUTPUT );
    pinMode( TYPING, OUTPUT );
    // Wired OR
    digitalWrite( LED_BUILTIN, on );
    digitalWrite( TYPING, LOW );
  } else {
    pinMode( LED_BUILTIN, OUTPUT );
    // to turn on a PULLUP make pin input
    pinMode( TYPING, INPUT_PULLUP );
    digitalWrite( LED_BUILTIN, on );
  }
}

void delay_( unsigned long ms ) {
  delay( ms );
}



void wait() {
  if ( digitalRead( SLOW ) == 1 ) {
    delay( random( 50, 250 ) );
  } else {
    delay( random( 0, 100 ) );
  }
}

void waitLong() {
  int count;
  delay( random( 1000, 2000 ) );
}

void incStringPtr() {
  unsigned long ts = 0;

  ptr ++ ;

  if ( ptr[0] == ' ') {
  }

  if ( ptr[0] == '\0') {
    ptr = myString;
    if ( slowSpeed ) {
      slowSpeed = false;
    } else {
      slowSpeed = true;
    } 
    ts = millis();
  }
}

void typeKey( char letter ) {
  if ( digitalRead( HALT ) == 1 ) {
    if ( isControl( letter ) && digitalRead( NOENTER ) ) {
      letter = ' ';
    }
    Serial1.print( letter );
    incStringPtr();
  }
}

void loop() {
   delay(1); 
  polledTimeSlice();
  if ( waiting ) {
    return;
  }

  if ( isControl( ptr[0] ) ) {
    LED( LOW );

    waiting = true;

    typeKey( ptr[0] );
    wait();
  } else {
    LED( HIGH );

    typeKey( ptr[0] );
    if ( slowSpeed ) {
      wait();
    }
    wait();
  }

  if ( isWhitespace( ptr[0] ) ) {
    wait();
  }
}

![This is an image](https://raw.githubusercontent.com/0GabrielGonzalez0/Gabriel_Gonzalez/main/Captura%20de%20pantalla%202022-11-17%20160024.png)



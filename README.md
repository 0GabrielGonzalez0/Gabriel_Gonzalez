# Gabriel_Gonzalez
#define TYPING_INTER_KEY_DELAY 20
//
// there was a bug with the SmartPhone App
// Pressing ENTER looses focus, so suppress ENTER
// select if "\n" presses ENTER key
//
#define nNOENTER true
#define NOENTER false

/*
   This code can use a Raspberry PICO or RP2
   Arduino IDE Tools configuration

   RP2 PICO
    Board: Raspberry Pi Pico
    Programmer: N/A - ensure  bootsel button is pressed when you plug in PICO
*/
/*
  #include "PluggableUSBHID.h"
  #include "USBKeyboard.h"

  USBKeyboard Keyboard;
*/

#define NOENTER 2
#define SLOW 6
#define HALT 9
#define TYPING 10
/*
  PICO pins:-

  USB
  (GP0)(GP1)[GND](GP2)(GP3)(GP4)(GP5)[GND](GP6)(GP7)(GP8)(GP9)[GND](GP10)(GP11)(GP12)(GP13)[GND](GP14)(GP15)

   Use links / slide switches to select a feature
   GP2  - SEND \N - send ENTER, it de-focused the input area, so allow it to be disabled.
   GP6  - SLOW    - strap low for faster typing
   GP9  - HALT    - The Pico can be reprogrammed using BOOTSEL, so this is not essential
   GP10 - TYPING  - assert low when typing, Only start typing if high.
*/

/*========================================================
   SEC: 2.0 - Typing Text
  ======================================================*/

#define PHONE
/* Phone User typing - The \n indicate end of sentence.*/
#ifdef PHONE
int phone = true;

// send space followed by a space.
// The phone looses focus when the  ENTER key is pressed.
// It buffers text and waits for a pause. send some single spaces or double spaces.
// A user could send GA to hand over to the other End
// A \n is used to signal end of typing.

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

/*========================================================
   SEC: 4.0 - Functions
  ======================================================*/

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

  // using unsigned so be caureful.
  if ( now - lastTs > 300 ) {
    // 0.1 second tick
    lastTs = now;

    //
    // payload - run every 300 ticks.
    //

    // if typing TYPING asserted Low, wait for it to go high
    // If input goes low turn off waiting
    //if ( ! digitalRead( IP ) ){
    if ( digitalRead( TYPING ) ) {
      waiting = false;
    }
  }
}

void LED( int on ) {
  /* IF on == TRUE then Typing. Assest TYPING LOW using Wired OR */

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

/*========================================================
   SEC: 5.0 - Functions -  delay
  ======================================================*/

void delay_( unsigned long ms ) {
  delay( ms );
}



void wait() {
  // pick a random delay and square it to make it more like real typing?
  //delay(random( 0, TYPING_INTER_KEY_DELAY )*random( 0, TYPING_INTER_KEY_DELAY ));
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

/*========================================================
   SEC: 6.0 - Functions Increment  & Type Key
  ======================================================*/

void incStringPtr() {
  unsigned long ts = 0;

  /* end of string return to start.*/
  ptr ++ ;

  // if space end of word.
  if ( ptr[0] == ' ') {
  }

  // END OF LOOP
  if ( ptr[0] == '\0') {
    ptr = myString;
    if ( slowSpeed ) {
      slowSpeed = false;
    } else {
      slowSpeed = true;
    } 
    ts = millis();
    /*
      Keyboard.printf( "%s", " ts: " );
      Keyboard.printf( "%d", ts / 1000 );
      Keyboard.printf( "%s", " seconds " );
    */
  }
}

void typeKey( char letter ) {
  // enable typing if HALT is high in this version.
  if ( digitalRead( HALT ) == 1 ) {
    // this is generally not necessary but with some older systems it seems to
    // prevent missing the first character after a delay:
    // DigiKeyboard.sendKeyStroke(0);
    // if ( NOENTER && isControl( letter ) ) {
    // Read GP2 to select sending \n
    if ( isControl( letter ) && digitalRead( NOENTER ) ) {
      letter = ' ';
    }
    //Keyboard.printf( "%c", letter );
    Serial1.print( letter );
    incStringPtr();
  }
}
/*


*/

/*========================================================
   SEC: 7.0 - Functions Main Loop
  ======================================================*/

void loop() {
   delay(1); 
  polledTimeSlice();
  /* work through string and print character by character */
  /* check if input low and enable typing. */
  if ( waiting ) {
    return;
  }

  // look for end of sentence,
  if ( isControl( ptr[0] ) ) {

    // turn off LED at end of sentence
    // This is used to tell other end as well.
    LED( LOW );

    // end of sentence turn os Waiting flag
    // so loop waits for other end to finish sentence.
    waiting = true;

    typeKey( ptr[0] );
    wait();
  } else {
    LED( HIGH );

    /* type character */
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


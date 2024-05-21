# utl-sharing-a-common-block-of-memory-with-dosubl-using-sas-peek-and-poke
Sharing a common block of memory with dosubl using sas peek and poke
    %let pgm=utl-sharing-a-common-block-of-memory-with-dosubl-using-sas-peek-and-poke;

      Sharing a common block of memory with dosubl using sas peek and poke;

      github
      https://tinyurl.com/5e36hmc3
      https://github.com/rogerjdeangelis/utl-sharing-a-common-block-of-memory-with-dosubl-using-sas-peek-and-poke

      Common macro on end

        Problem
            Change data in a shared common block between parent
            datastep and dosubl.
            I am pretty sure dosubl shares the address space
            with the parent datastep.

        HOW IN WORKS

           data parent;

               retain cartype "ACCURA";
               put cartype=;  /* result cartype=ACCURA             */

               %commonc(cartype $8,ACTION=INIT);
               /* creates global macro var with adr(cartype)       */
               /* for example "201EEBEDF7010000"x                  */

               rc=dosubl('
                 /* global adr contains 201EEBEDF7010000 from init */
                 cartype="HONDA";                                  */
                 %commonc(cartype $8, ACTION=PUT);
                 /* call pokelong(cartype,"201EEBEDF7010000"x, 8); */
               ');

               put cartype=; /* result cartype=HONDA               */

            run;quit;

    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*      INPUT                                PROCESS (update parent memory)              OUTPUT                           */
    /*                                                                                                                        */
    /*  Common block of storage contain          data _null_;                                                                 */
    /*                                            %commonc(cartype $8,action=INIT);         Shared common block of storage    */
    /*  cartype='ACURA';                          cartype='ACURA';                          updated by dosubl                 */
    /*  %commonc(cartype $8,action=INIT);         put cartype=; /* CARTYPE=ACURA */         now parent datastep has           */
    /*                                             rc=dosubl('                                                                */
    /*  put cartype=; * CARTYPE=ACURA                data _null_;                           put cartype=; * CARTYPE=HONDA     */
    /*                                                   cartype="HONDA";                                                     */
    /*                                                   %commonc(cartype $8,ACTION=PUT);                                     */
    /*                                                run;                                                                    */
    /*                                           ');                                                                          */
    /*                                            put cartype=; /* CARTYPE=HONDA */                                           */
    /*                                           run;quit;                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

    data _null_;
     %commonc(cartype $8,action=INIT);
     cartype='ACURA';
     put cartype=; /* CARTYPE=ACURA */
      rc=dosubl('
         data _null_;
            cartype="HONDA";
            %commonc(cartype $8,ACTION=PUT);
         run;
    ');
     put cartype=; /* CARTYPE=HONDA */
    run;quit;

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  Parent                                                                                                                */
    /*   CARTYPE=ACURA                                                                                                        */
    /*                                                                                                                        */
    /*  Dosubl                                                                                                                */
    /*   CARTYPE=HONDA                                                                                                        */
    /*                                                                                                                        */
    /*  Back to Parent                                                                                                        */
    /*   CARTYPE=HONDA                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

     /*
     _ __ ___   __ _  ___ _ __ ___
    | `_ ` _ \ / _` |/ __| `__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    */

    %macro commonc(var,action=INIT);
     * dosubl sets sysindex to 1;
     * we are in dosubl if sysindex=1;
     * increment sysindex so it is not 1 next time macro called;
     %local varcut varlen;

     %let varcut=%scan(&var,1);
     %let varlen=%scan(&var,2);

     %if %upcase(&action) = INIT %then %do;
        length &var;
        retain &varcut " ";
        call symputx("varadr",put(addrlong(&varcut.),hex16.),"G");
        put "***PARENT &var &varcut &varlen &SYSDATASTEPPHASE &sysindex";
        initadr=put(addrlong(&varcut.),hex16.);
        put initadr;
     %end;
     %if "%upcase(&action)" = "PUT" %then %do;
        length &var;
        retain &varcut;
        call pokelong(&varcut.,"&varadr."x, &varlen.);
     %end;
     %else %if "%upcase(&action)" = "GET" %then %do;

        retain &varcut " ";
        &varcut = peekclong("&varadr."x,&varlen.);
        %end;
        put "***CHILD &var &varcut &varlen &SYSDATASTEPPHASE &sysindex";
    %mend commonc;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

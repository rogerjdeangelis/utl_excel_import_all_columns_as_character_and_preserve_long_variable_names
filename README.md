# utl_excel_import_all_columns_as_character_and_preserve_long_variable_names
Excel import all columns as character and preserve long variable names. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Excel import all columns as character and preserve long variable names

     I suggest you bring all the data into SAS as character data and fix it on the SAS side.

     You can use IML/R
     utl_rens macro on end

     R is a bit slow, i suspect r reads the sheets twice to get lengths?

    github
    https://tinyurl.com/ydb98fjx
    https://github.com/rogerjdeangelis/utl_excel_import_all_columns_as_character_and_preserve_long_variable_names

    see SAS Forum
    https://tinyurl.com/y6twrqlu
    https://communities.sas.com/t5/SAS-Programming/Further-process-after-PROC-IMPORT/m-p/497835

    related respositories
    https://tinyurl.com/ycjcrvas
    https://github.com/rogerjdeangelis/utl_R_sas_v5_xport_with_long_variable_names
    https://tinyurl.com/yd5blldf
    https://github.com/rogerjdeangelis/utl_proc_import_columns_as_character_from_excel_linux_or_windows


    INPUT
    =====

      * this create 3 workbooks in d:/xls;

         d:/xls/class.xlsx
         d:/xls/cars.xlsx
         d:/xls/classfit.xlsx


    EXAMPLE OUTPUT (All columns are character)
    ------------------------------------------


     WORK.CLASSFIT_CHR

      Variables in Creation Order

       #    Variable     Type    Len    Label

       1    Name         Char      7    Name
       2    Sex          Char      1    Sex
       3    Age          Char      2    Age
       4    Height       Char      4    Height
       5    Weight       Char      5    Weight
       6    predict      Char     16    predict
       7    lowermean    Char     16    lowermean   ** note the long names
       8    uppermean    Char     16    uppermean   ** note the long names
       9    lower        Char     16    lower
      10    upper        Char     16    upper


     WORK.CARS_CHR

                Variables in Creation Order

       #    Variable       Type    Len    Label

       1    Make           Char     13    Make
       2    Model          Char     40    Model
       3    Type           Char      6    Type
       4    Origin         Char      6    Origin
       5    DriveTrain     Char      5    DriveTrain
       6    MSRP           Char      6    MSRP
       7    Invoice        Char      6    Invoice
       8    EngineSize     Char      3    EngineSize
       9    Cylinders      Char      2    Cylinders
       0    Horsepower     Char      3    Horsepower
       1    MPG_City       Char      2    MPG_City
       2    MPG_Highway    Char      2    MPG_Highway
       3    Weight         Char      4    Weight
       4    Wheelbase      Char      3    Wheelbase
       5    Length         Char      3    Length

     WORK.CLASS_CHR

             Variables in Creation Order

       #    Variable    Type    Len    Label

       1    Name        Char      7    Name
       2    Sex         Char      1    Sex
       3    Age         Char      2    Age
       4    Height      Char      4    Height
       5    Weight      Char      5    Weight

    PROCESS
    =======

    data _null_;

      do fid="classfit", "class", "cars";

        call symputx("fid",fid);

         rc=dosubl('
          %utl_submit_r64(resolve(''
           library(xlsx);
           library(Hmisc);
           library(SASxport);
           &fid.<-read.xlsx("d:/xls/&fid..xlsx",1,colClasses=rep("character",16),stringsAsFactors=FALSE);
           for (i in seq_along(&fid.)) {
              label(&fid.[,i])<- colnames(&fid.)[i];
           };
           str(&fid);
           write.xport(&fid.,file="d:/xpt/rxpt.xpt");
         ''))
         libname xpt xport "d:/xpt/rxpt.xpt";

         data &fid._chr;

           %utl_rens(xpt.&fid.);
           set &fid.;

         run;quit;
       ');

      end;

    run;quit;

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;
    %utlfkil(d:/xls/class.xlsx);    * delete if exist;
    %utlfkil(d:/xls/cars.xlsx);
    %utlfkil(d:/xls/classfit.xlsx);

    libname class    "d:/xls/class.xlsx";
    libname cars     "d:/xls/cars.xlsx";
    libname classfit "d:/xls/classfit.xlsx";

    data class.class    ; set sashelp.class    ; run;quit;
    data cars.cars     ; set sashelp.cars     ; run;quit;
    data classfit.classfit ; set sashelp.classfit ; run;quit;

    libname class    clear;
    libname cars     clear;
    libname classfit clear;

    *      _   _
     _   _| |_| |    _ __ ___ _ __  ___
    | | | | __| |   | '__/ _ \ '_ \/ __|
    | |_| | |_| |   | | |  __/ | | \__ \
     \__,_|\__|_|___|_|  \___|_| |_|___/
               |_____|
    ;

    %macro utl_rens(dsn);

      if _n_=0 then do;
        rc=%sysfunc(dosubl('

            data __ren001;
               set &dsn(obs=1);
            run;quit;

            proc transpose data=__ren001 out=__ren002(drop=col1);
              var _all_;
            run;quit;

            proc sql;
              select
                catx(' ',_name_,"as",lbl) into :rens separated by ","
              from
                (
                 select
                    _name_
                   ,case
                        when (_label_ = ' ') then _name_
                        else _label_
                    end as lbl
                 from
                    __ren002
                )
           ;quit;

            proc sql;
               create
                   view %scan(&dsn,2,'.')  as
               select
                   &rens
               from
                   &dsn.
            ;quit;
        '));
        drop rc;
      end;

    %mend utl_rens;


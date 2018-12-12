# utl-classic-problem-with-proc-transpose-and-mutiple-variables-seven-solutions
Classic problem with proc transpose and mutiple variables seven solutions  
    Classic problem with proc transpose and mutiple variables seven solutions                                                            
                                                                                                                                         
      Seven Solutions                                                                                                                    
                                                                                                                                         
        1. Utl_transpose macro (preferred-very fast Art Tabachneck)                                                                      
        2. SQL arrays (good for in database awspecially teradata snd exadata)                                                            
        3. Proc report (most flexible sort,transpose, summarise and datastep like processing)                                            
        4. Summary id group                                                                                                              
        5. Double transpose (not preferred?)                                                                                             
        6. Utl_gather (Alea Iacta) https://github.com/clindocu                                                                           
        7. R data table                                                                                                                  
                                                                                                                                         
    github                                                                                                                               
    https://tinyurl.com/yd7gawmn                                                                                                         
    https://github.com/rogerjdeangelis/utl-classic-problem-with-proc-transpose-and-mutiple-variables-seven-solutions                     
                                                                                                                                         
    Transpose repositories                                                                                                               
    https://github.com/rogerjdeangelis?utf8=%E2%9C%93&tab=repositories&q=transpose+in%3Aname&type=&language=                             
                                                                                                                                         
    utl_Transpose macro by                                                                                                               
    AUTHORS: Arthur Tabachneck, Xia Ke Shan, Robert Virgile and Joe Whitehurst                                                           
                                                                                                                                         
    Macros                                                                                                                               
    https://tinyurl.com/y9nfugth                                                                                                         
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories                                           
                                                                                                                                         
                                                                                                                                         
    Wimpel Profile                                                                                                                       
    https://stackoverflow.com/users/6356278/wimpel                                                                                       
                                                                                                                                         
    https://tinyurl.com/yb9rhtty                                                                                                         
    https://stackoverflow.com/questions/53664307/is-there-a-better-way-to-spread-a-long-table-with-mutlple-columns-into-a-wide           
                                                                                                                                         
                                                                                                                                         
    PROCESS                                                                                                                              
    =======                                                                                                                              
                                                                                                                                         
    1. Utl_transpose macro (preferred-very fast )                                                                                        
       -------------------------------------------                                                                                       
                                                                                                                                         
       %utl_transpose(data=have, out=want_rpt, by=file, id=label,                                                                        
             delimiter=_, var=val1 val2, var_first=no);                                                                                  
                                                                                                                                         
       /*                                                                                                                                
       WORK.WANT_RPT total obs=3                                                                                                         
                                                                                                                                         
        FILE     A_VAL1    A_VAL2    B_VAL1    B_VAL2    C_VAL1    C_VAL2                                                                
                                                                                                                                         
        green       3         3         6         5         9         6                                                                  
        red        12         3         4         2         5         8                                                                  
        blue        3         3         1         2         4         6                                                                  
       */                                                                                                                                
                                                                                                                                         
    2. SQL Arrays                                                                                                                        
       ----------                                                                                                                        
                                                                                                                                         
                                                                                                                                         
         %array(lbls,values=A B C);                                                                                                      
                                                                                                                                         
         proc sql;                                                                                                                       
            create                                                                                                                       
               table want as                                                                                                             
            select                                                                                                                       
               file                                                                                                                      
              ,%do_over(lbls, phrase=%str(                                                                                               
                  sum((label="?lbls")*val1) as value1_?lbls),between=comma)                                                              
              ,%do_over(lbls, phrase=%str(                                                                                               
                  sum((label="?lbls")*val2) as value2_?lbls),between=comma)                                                              
            from                                                                                                                         
                have                                                                                                                     
            group                                                                                                                        
                by file                                                                                                                  
         ;quit;                                                                                                                          
                                                                                                                                         
         /*                                                                                                                              
         WORK.WANT total obs=3                                                                                                           
                                                                                                                                         
         FILE   VALUE1_A    VALUE1_B    VALUE1_C    VALUE2_A    VALUE2_B    VALUE2_C                                                     
                                                                                                                                         
         blue       3           1           4           3           2           6                                                        
         green      3           6           9           3           5           6                                                        
         red       12           4           5           3           2           8                                                        
        */                                                                                                                               
                                                                                                                                         
    3.  /* SAS should honor ODS output? */                                                                                               
    --------------------------------------                                                                                               
                                                                                                                                         
        proc report data=have out=want_rpt(drop=_break_)                                                                                 
         ( rename=(  /* SAS should honor ODS output so we do not need this */                                                            
             _C2_ = A_VAL1                                                                                                               
             _C3_ = A_VAL2                                                                                                               
             _C4_ = B_VAL1                                                                                                               
             _C5_ = B_VAL2                                                                                                               
             _C6_ = C_VAL1                                                                                                               
             _C7_ = C_VAL2                                                                                                               
         )) nowd missing;                                                                                                                
       cols file label, ( val1 val2 );                                                                                                   
       define file / group;                                                                                                              
       define val1 / analysis sum;                                                                                                       
       define val2 /analysis sum;                                                                                                        
       define label / across;                                                                                                            
       run;quit;                                                                                                                         
                                                                                                                                         
       /*                                                                                                                                
       WANT_RPT total obs=3                                                                                                              
                                                                                                                                         
       FILE     A_VAL1    A_VAL2    B_VAL1    B_VAL2    C_VAL1    C_VAL2                                                                 
                                                                                                                                         
       blue        3         3         1         2         4         6                                                                   
       green       3         3         6         5         9         6                                                                   
       red        12         3         4         2         5         8                                                                   
       */                                                                                                                                
                                                                                                                                         
                                                                                                                                         
    4. Summary id group                                                                                                                  
    -------------------                                                                                                                  
                                                                                                                                         
       proc summary data=have;                                                                                                           
       by file notsorted;                                                                                                                
       output idgroup(out[3] (val1 val2)=)                                                                                               
          out=want (drop=_type_ _freq_  rename=(                                                                                         
            VAL1_1 = VALUE1_A                                                                                                            
            VAL1_2 = VALUE1_B                                                                                                            
            VAL1_3 = VALUE1_C                                                                                                            
            VAL2_1 = VALUE2_A                                                                                                            
            VAL2_2 = VALUE2_B                                                                                                            
            VAL2_3 = VALUE2_C));                                                                                                         
                                                                                                                                         
       run;quit;                                                                                                                         
                                                                                                                                         
       /*                                                                                                                                
       WORK.WANT total obs=3                                                                                                             
                                                                                                                                         
       FILE     VALUE1_A    VALUE1_B    VALUE1_C    VALUE2_A    VALUE2_B    VALUE2_C                                                     
                                                                                                                                         
       green        3           6           9           3           5           6                                                        
       red         12           4           5           3           2           8                                                        
       blue         3           1           4           3           2           6                                                        
       */                                                                                                                                
                                                                                                                                         
                                                                                                                                         
    5. Double transpose (not preferred?)                                                                                                 
    -------------------------------------                                                                                                
                                                                                                                                         
       proc transpose data=have out=havXpo;                                                                                              
       by file label notsorted;                                                                                                          
       var val1 val2 ;                                                                                                                   
       run;                                                                                                                              
                                                                                                                                         
       FILE     LABEL    _NAME_    COL1                                                                                                  
                                                                                                                                         
       green      A       VAL1       3                                                                                                   
       green      A       VAL2       3                                                                                                   
       green      B       VAL1       6                                                                                                   
                                                                                                                                         
       proc transpose data=havXpo out=havXpoDuo(drop=_name_);                                                                            
       by file notsorted;                                                                                                                
       id _name_ label;                                                                                                                  
       var col1;                                                                                                                         
       run;quit;                                                                                                                         
                                                                                                                                         
       /*                                                                                                                                
       HAVXPODUO total obs=3                                                                                                             
                                                                                                                                         
        FILE     VAL1A    VAL2A    VAL1B    VAL2B    VAL1C    VAL2C                                                                      
                                                                                                                                         
        green       3       3        6        5        9        6                                                                        
        red        12       3        4        2        5        8                                                                        
        blue        3       3        1        2        4        6                                                                        
       */                                                                                                                                
                                                                                                                                         
                                                                                                                                         
    6. Utl_gather (Alea Iacta)                                                                                                           
    --------------------------                                                                                                           
                                                                                                                                         
       %utl_gather(have,var,val,file label,xpoGat,valformat=3.);                                                                         
                                                                                                                                         
        /*                                                                                                                               
        FILE     LABEL    VAR     VAL                                                                                                    
                                                                                                                                         
        green      A      VAL1      3                                                                                                    
        green      A      VAL2      3                                                                                                    
        green      B      VAL1      6                                                                                                    
        green      B      VAL2      5                                                                                                    
        */                                                                                                                               
                                                                                                                                         
       proc transpose data=xpoGat out=havXpoGat(drop=_name_);                                                                            
       by file notsorted;                                                                                                                
       id var label;                                                                                                                     
       var val;                                                                                                                          
       run;quit;                                                                                                                         
                                                                                                                                         
       /*                                                                                                                                
       WORK.HAVXPOGAT total obs=3                                                                                                        
                                                                                                                                         
       FILE     VAL1A    VAL2A    VAL1B    VAL2B    VAL1C    VAL2C                                                                       
                                                                                                                                         
       green       3       3        6        5        9        6                                                                         
       red        12       3        4        2        5        8                                                                         
       blue        3       3        1        2        4        6                                                                         
       */                                                                                                                                
                                                                                                                                         
                                                                                                                                         
    7. R data table                                                                                                                      
    ---------------                                                                                                                      
                                                                                                                                         
      options validvarname=upcase;                                                                                                       
      libname sd1 "d:/sd1";                                                                                                              
      data sd1.have;                                                                                                                     
       input file$ label$ val1 val2;                                                                                                     
      cards4;                                                                                                                            
      green A 3 3                                                                                                                        
      green B 6 5                                                                                                                        
      green C 9 6                                                                                                                        
      red A 12 3                                                                                                                         
      red B 4 2                                                                                                                          
      red C 5 8                                                                                                                          
      blue A 3 3                                                                                                                         
      blue B 1 2                                                                                                                         
      blue C 4 6                                                                                                                         
      ;;;;                                                                                                                               
      run;quit;                                                                                                                          
                                                                                                                                         
      %utl_submit_r64('                                                                                                                  
      library(haven);                                                                                                                    
      library(tidyr);                                                                                                                    
      library(dplyr);                                                                                                                    
      library(SASxport);                                                                                                                 
      have<-read_sas("d:/sd1/have.sas7bdat");                                                                                            
      want<-have %>%                                                                                                                     
        gather("variable_name", "value", contains("VAL")) %>%                                                                            
        mutate(new_column_name = paste(variable_name, LABEL, sep="_")) %>%                                                               
        select(-LABEL, -variable_name) %>%                                                                                               
        spread(new_column_name, value);                                                                                                  
      want<-as.data.frame(want);                                                                                                         
      write.xport(want,file="d:/xpt/want.xpt");                                                                                          
      ');                                                                                                                                
                                                                                                                                         
      /*                                                                                                                                 
      WORK.WANT total obs=3                                                                                                              
                                                                                                                                         
         FILE     VAL1_A    VAL1_B    VAL1_C    VAL2_A    VAL2_B    VAL2_C                                                               
                                                                                                                                         
         blue        3         1         4         3         2         6                                                                 
         green       3         6         9         3         5         6                                                                 
         red        12         4         5         3         2         8                                                                 
      */                                                                                                                                 
                                                                                                                                         
                                                                                                                                         
    *                _              _       _                                                                                            
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _                                                                                     
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |                                                                                    
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |                                                                                    
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|                                                                                    
                                                                                                                                         
    ;                                                                                                                                    
                                                                                                                                         
    data have;                                                                                                                           
     input file$ label$ val1 val2;                                                                                                       
    cards4;                                                                                                                              
    green A 3 3                                                                                                                          
    green B 6 5                                                                                                                          
    green C 9 6                                                                                                                          
    red A 12 3                                                                                                                           
    red B 4 2                                                                                                                            
    red C 5 8                                                                                                                            
    blue A 3 3                                                                                                                           
    blue B 1 2                                                                                                                           
    blue C 4 6                                                                                                                           
    ;;;;                                                                                                                                 
    run;quit;                                                                                                                            
                                                                                                                                         
                                                                                                                                         
                                                                                                                                         

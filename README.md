# utl-identify-linked-and-unliked-paths-r-igraph
    %let pgm=utl-identify-linked-and-unliked-paths-r-igraph;

    Identify linked and unliked paths r igraph

    https://github.com/rogerjdeangelis/utl-identify-linked-and-unliked-paths-r-igraph

    github
    https://tinyurl.com/4j46braf
    https://stackoverflow.com/questions/77059053/linking-records-with-record-expansion-in-r

    output linked graph
    https://tinyurl.com/4n2hm2p6
    https://github.com/rogerjdeangelis/utl-identify-linked-and-unliked-paths-r-igraph/blob/main/igraph.pdf

     Solutions

       1 R
       2 related repos

    /**************************************************************************************************************************/
    /*             |                            |                                                                             */
    /*  INPUT    m | RULES                      | OUTPUT                   LINKED and UNLINKED PATHS                          */
    /*             |                            |                                                                             */
    /*   FRO  TOO  |                            |PATHS FRO  TOO       ---+--------+---------+-------+--------+--              */
    /*             | PATH 1                     |                     |                                        |              */
    /*    1   123  | 1->123-123<-2->125-125 <-3 |  1    1   123     2 +             4 --> 200                 +               */
    /*    2   123  |                            |  1    2   123       |                                        |              */
    /*    2   125  |                            |  1    2   125       |                 Transitive             |              */
    /*    3   125  |                            |  1    3   125     1 +  1  ---> 123  <--- 2 --->  125 <---  3 +              */
    /*             | PATH 2                     |                     |                                        |              */
    /*    4   200  | 4 -> 200                   |  4    4   200       ---+--------+---------+-------+--------+--              */
    /*                                                                                                                        */
    /*                                                                https://tinyurl.com/4n2hm2p6  (hi res graph)            */
    /*                                                                                                                        */
    /*                                                                                                                        */
    /* 'If we plot the graph, we will see that there are two unconnected clusters of nodes                                    */
    /* (called components); one for the grouped IDs 1:3, and one for ID 4'                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
      input fro too;
    cards4;
    1   123
    2   123
    2   125
    3   125
    4   200
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* SD1.HAVE total obs=5                                                                                                   */
    /*                                                                                                                        */
    /* Obs    ID     X                                                                                                        */
    /*                                                                                                                        */
    /*  1      1    123                                                                                                       */
    /*  2      2    123                                                                                                       */
    /*  3      2    125                                                                                                       */
    /*  4      3    125                                                                                                       */
    /*  5      4    200                                                                                                       */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*   ____
    / | |  _ \
    | | | |_) |
    | | |  _ <
    |_| |_| \_\

    */

    %utlfkil("d:/pdf/igraph.pdf");

    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    library(sqldf);
    library(igraph);
    library(tidyverse);
    g <- graph_from_data_frame(have);
    pdf("d:/pdf/igraph.pdf",width = 5, height =4);
    plot(g);
    pdf();
    comp <- components(g)$membership;
    want<-have %>%
      mutate(group = comp[match(FRO, names(comp))]) %>%
      mutate(id = first(FRO), .by = "group") %>%
      select(-group);
    want;
    endsubmit;
    import data=sd1.want r=want;
    run;quit;

    proc print data=sd1.want width=min;
    run;quit;

    ');

    data froToo;
     input x y txt;
    cards4;
    1.0 1 1
    1.5 1 123
    2.0 1 2
    2.5 1 125
    3.0 1 3
    1.5 2 4
    1.5 2 200
    ;;;;
    run;quit;

    options ls=64 ps=24;
    proc plot data=frotoo;
     plot y*x ='<' $ txt /box haxis=1 to 4;
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
    /*  The WPS System      data froToo;         You nedd to do some editiing                                                 */
    /*                       input x y txt;                                                                                   */
    /*   FRO    TOO    ID   cards4;                ---+--------+---------+-------+--------+--                                 */
    /*                      1.0 1 1                |                                        |                                 */
    /*    1     123     1   1.5 1 123            2 +             200-->4                    +                                 */
    /*    2     123     1   2.0 1 2                |                                        |                                 */
    /*    2     125     1   2.5 1 125              |                 Transitive             |                                 */
    /*    3     125     1   3.0 1 3              1 +  1  ---> 123  <--- 2 --->  125 <---  3 +                                 */
    /*    4     200     4   1.5 2 4                |                                        |                                 */
    /*                      1.5 2 200              ---+--------+---------+-------+--------+--                                 */
    /*                      ;;;;                                                                                              */
    /*                      run;quit;                                                                                         */
    /*                                                                                                                        */
    /*                      options ls=64 ps=15;                                                                              */
    /*                      proc plot data=frotoo;                                                                            */
    /*                       plot y*x ='<' $ txt /box haxis=1 to 4;                                                           */
    /*                      run;quit;                                                                                         */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___             _       _           _
    |___ \   _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
      __) | | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
     / __/  | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
    |_____| |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                      |_|
    */

    https://github.com/rogerjdeangelis/utl-R-AI-igraph-list-connections-in-a-non-directed-graph-for-a-subset-of-vertices
    https://github.com/rogerjdeangelis/utl-how-many-triangles-in-the-polygon-r-igraph-AI
    https://github.com/rogerjdeangelis/utl-shortest-and-longest-travel-time-from-home-to-work-igraph-AI
    https://github.com/rogerjdeangelis/utl_remove_isolated_nodes_from_an_network_r_igraph

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
Identify linked and unliked paths r igraph  

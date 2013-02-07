Scribe
=========

Scribe is a program written in Python to help the creation of mapfiles with different scale levels. Managing the scale levels
is made easy with scale tags that can be nested inside any regular mapfile tag. Also, the creation of mapfiles becomes much faster 
with the use of variables.

Utilisation
-----------

python decoder.py -n outputmapname
       
    Optional parameters
    -------------------
        -n/--name: (default 'result') Name of the output map
        -i/--input: (default ./) Input directory
        -o/--output: (default ./result/) Output directory
        -t/--tabulation: (default 4) Number of spaces to use as tabulation
        -f/--file: (default config) Config file. Path can be absolute or relative to the scribe.py file.

    Other options
    -------------
        -c/--clean: If this option is specified, the layers from each scale level are included in the output mapfile with 'INCLUDE' tags. If not specified, the content of each scale level is outputted directly in the resulting mapfile.  
    
Input
-----

The following files should be present in the input directory <br />
    * scales: contains the scale levels and denominators (MINSCALEDENOM <br />
    * variables: contains variables that can be called as follow : @myvariable <br />
    * map: contains everything inside the MAP tag that is not a LAYER <br />
    * Any ".layer" file: contain the layers
    * Any config file: indicates the order in which the ".layer" files should be included

    Example for file 'scales'
    -------------------------
        SCALES {
            1:268435456
            2:134217728
            3:67108864
            4:33554432
            5:16777216
            6:8388608
            7:4194304
            8:2097152
            9:1048576
            10:524288
            11:262144
            12:131072
            13:65536
            14:32768
            15:16384
            16:8192
        }

    Example for file 'variables'
    ----------------------------
        VARIABLES {
            connection {
                CONNECTION: "host=localhost dbname=mydb user=username password=pword port=5432"
                CONNECTIONTYPE: POSTGIS
            }
            layerconfig {
                GROUP: 'default'
                STATUS: ON
                PROJECTION {{
                    'init=epsg:4326'
                }}
                PROCESSING: 'LABEL_NO_CLIP=ON'
                PROCESSING: 'CLOSE_CONNECTION=DEFER'
            }
            water_clr: '#C6E2F2'
            land_ol_width {
                1-4: 1
                5-8: 0.5 
            }    
        }

        * Note the use of : between the parameters and their value.

        * For the PROJECTION tag, {{}} are used instead of {} because PROJECTION contains no parameter, only plain text that should be outputted as it is.
          Same goes for PATTERN, METADATA etc.

    Example for file 'map'
    ----------------------
        MAP {
            CONFIG: 'PROJ_LIB' '../'
            FONTSET: '../fonts.lst'
            IMAGETYPE: png
            MAXSIZE: 4000
            SIZE: 800 800
            UNITS: meters
            EXTENT: -20000000 -20000000 20000000 20000000
            IMAGECOLOR: @water_clr
            WEB {
                METADATA {{
                    "ows_enable_request" "*"
                    "wms_srs" "EPSG:900913 EPSG:4326"
                    "labelcache_map_edge_buffer" "-10"
                    "wms_title" "Natural Earth World WMS"
                }}
                IMAGEPATH: '/tmp/ms_tmp/'
                IMAGEURL: '/ms_tmp/'
            }
            PROJECTION {{
                "init=epsg:900913"
            }}
        }

        * IMAGECOLOR will take the value  of the 'water_clr' variable

    Example of a '.layer' file
    ----------------------------
        LAYER {
            1-16 {
                NAME: 'land'
                TYPE: POLYGON
                @layerconfig
                DATA {
                    1-4: '110m_physical/ne_110m_land'
                    5-10: '50m_physical/ne_50m_land'
                    11-16: '10m_physical/ne_10m_land'
                }
                CLASS {
                    STYLE {
                        COLOR: '#EEECDF'
                        OUTLINECOLOR: 200 200 200
                        OUTLINEWIDTH: @land_ol_width 
                    }
                }
            }
        }
        ## This is a single line comment that will be outputted
        /*This is a multiline comment 
        that won't be outputted*/
        LAYER {
            4-16 {
                NAME: 'lakes' //Another type of single line comment that won't be outputted
                TYPE: POLYGON
                @layerconfig
                DATA {
                    1-4: '110m_physical/ne_110m_lakes'
                    5-15: '50m_physical/ne_50m_lakes'
                    16: '10m_physical/ne_10m_lakes'
                }
                CLASS {
                    STYLE {
                        COLOR: @water_clr
                    }
                }
            }
        }

        * The land layer will be displayed from scale level 1 to 16 and the lakes layer will be displayed from scale level 4 to 16

        * The variable 'layerconfig' which contains many parameters used in every layer is called, which means those parameters need to be written only once.

        * The data for both layer changes according to the scale level which makes using generalized data easy. Even though the data for the lakes layer is specified for levels 1 to 3, the
          layer will be displayed only for level 4 to 16.

        * The parameter OUTLINEWIDTH of the land layer takes the value of a variable which itself contains scale dependant values.

        * Single line comment can be written using double # (##) and will be outputted in the resulting mapfile. They cannot be on the same line as text uncommented.
          Single line comments can also be written using //. Those type of comment won't be outputted in the resulting mapfile and can be placed anywhere.
          Multiline comments can be written using /* */ but won't be outputted in the resulting mapfile.

Output
------

The output is a complete mapfile as well as one mapfile per scale level (which can be used with the -c/--clean option)

    Example of a complete mapfile (MAP header and first 2 levels only)
    ------------------------------------------------------------------
    MAP
        CONFIG 'PROJ_LIB' '../../'
        FONTSET '../../fonts.lst'
        IMAGETYPE png
        MAXSIZE 4000
        SIZE 800 800
        UNITS meters
        EXTENT -20000000 -20000000 20000000 20000000
        IMAGECOLOR '#C6E2F2'
        SHAPEPATH '../../data/'
        WEB
            METADATA
                "ows_enable_request" "*"
                "wms_srs" "EPSG:900913 EPSG:4326"
                "labelcache_map_edge_buffer" "-10"
                "wms_title" "Natural Earth World WMS"
            END
            IMAGEPATH '/tmp/ms_tmp/'
            IMAGEURL '/ms_tmp/'
        END
        PROJECTION
            "init=epsg:900913"
        END     
    #---- LEVEL 1 ----#
        LAYER
            MAXSCALEDENOM 999999999
            MINSCALEDENOM 268435456
            NAME 'land1'
            TYPE POLYGON
            GROUP 'default'
            STATUS ON
            PROJECTION
                'init=epsg:4326'
            END
            PROCESSING 'LABEL_NO_CLIP=ON'
            PROCESSING 'CLOSE_CONNECTION=DEFER'
            DATA '110m_physical/ne_110m_land'
            CLASS
                STYLE
                    COLOR '#EEECDF'
                    OUTLINECOLOR 200 200 200
                    OUTLINEWIDTH 1
                END
            END
        END    
    ## This is a single line comment that will be outputted

    #---- LEVEL 2 ----#
        LAYER
            MAXSCALEDENOM 268435456
            MINSCALEDENOM 134217728
            NAME 'land2'
            TYPE POLYGON
            GROUP 'default'
            STATUS ON
            PROJECTION
                'init=epsg:4326'
            END
            PROCESSING 'LABEL_NO_CLIP=ON'
            PROCESSING 'CLOSE_CONNECTION=DEFER'
            DATA '110m_physical/ne_110m_land'
            CLASS
                STYLE
                    COLOR '#EEECDF'
                    OUTLINECOLOR 200 200 200
                    OUTLINEWIDTH 1
                END
            END
        END 
    ## This is a single line comment that will be outputted

    * The map header and the first two levels are exactly as expected. MAXSCALEDENOM and MINSCALEDENOM are automatically added and the scale level is appended to the layer name. At those
      levels, only land layer are expected to be displayed, which is the case.
    
    * The content of the variable 'layerconfig' as been properly written in every layer.

    * The mapfile is properly indented (-t/--tabulation option)

    * The single line comments (##) are repeated.


    Example of a complete mapfile (levels 4 and 5 only)
    ---------------------------------------------------    
    #---- LEVEL 4 ----#
        LAYER
            MAXSCALEDENOM 67108864
            MINSCALEDENOM 33554432
            NAME 'land4'
            TYPE POLYGON
            GROUP 'default'
            STATUS ON
            PROJECTION
                'init=epsg:4326'
            END
            PROCESSING 'LABEL_NO_CLIP=ON'
            PROCESSING 'CLOSE_CONNECTION=DEFER'
            DATA '110m_physical/ne_110m_land'
            CLASS
                STYLE
                    COLOR '#EEECDF'
                    OUTLINECOLOR 200 200 200
                    OUTLINEWIDTH 1
                END
            END
        END
    ## This is a single line comment that will be outputted
        LAYER
            MAXSCALEDENOM 67108864
            MINSCALEDENOM 33554432
            NAME 'lakes4'
            TYPE POLYGON
            GROUP 'default'
            STATUS ON
            PROJECTION
                'init=epsg:4326'
            END
            PROCESSING 'LABEL_NO_CLIP=ON'
            PROCESSING 'CLOSE_CONNECTION=DEFER'
            DATA '110m_physical/ne_110m_lakes'
            CLASS
                STYLE
                    COLOR '#C6E2F2'
                END
            END
        END

    #---- LEVEL 5 ----#
        LAYER
            MAXSCALEDENOM 33554432
            MINSCALEDENOM 16777216
            NAME 'land5'
            TYPE POLYGON
            GROUP 'default'
            STATUS ON
            PROJECTION
                'init=epsg:4326'
            END
            PROCESSING 'LABEL_NO_CLIP=ON'
            PROCESSING 'CLOSE_CONNECTION=DEFER'
            DATA '50m_physical/ne_50m_land'
            CLASS
                STYLE
                    COLOR '#EEECDF'
                    OUTLINECOLOR 200 200 200
                    OUTLINEWIDTH 0.5
                END
            END
        END
    ## This is a single line comment that will be outputted
        LAYER
            MAXSCALEDENOM 33554432
            MINSCALEDENOM 16777216
            NAME 'lakes5'
            TYPE POLYGON
            GROUP 'default'
            STATUS ON
            PROJECTION
                'init=epsg:4326'
            END
            PROCESSING 'LABEL_NO_CLIP=ON'
            PROCESSING 'CLOSE_CONNECTION=DEFER'
            DATA '50m_physical/ne_50m_lakes'
            CLASS
                STYLE
                    COLOR '#C6E2F2'
                END
            END
        END

        * At level 4, the lakes layer is now displayed.
     
        * At level 5, the data for both layer changes from 110m to 50m, which is less generalized.

Installation
------------

    Requirements
    ------------
    1) Python 2.x installed. Tested with 2.7 only but should work with previous versions.
    
    2) Any version of Mapserver.

That's it.

Getting started
---------------

    1) Four files named 'scales', 'variables', 'map' and 'layers' need to be in the same input directory. You should use the files given in example and modify them.

    2) Defining the scale levels in generally the first step.

    3) To produce a mapfile use the command explained in the 'Utilisation' section. 
    

Documentation
-------------

    {}: 1) Signifies the start and the end of a Mapserver block tag (LAYER, STYLE, CLASS etc.) as well as a scale block tag (1-10, 11 etc.).
        
        2) You can use them with single line tags to signify that this tag takes different value following the scale level (see DATA tag in the examples above)
    
    {{}}: Signifies the start and the end of a Mapserver block tag as well as a scale block the tag. The difference  with {} is that those block tags contain
        no parameter (PROJECTION, METADATA, PATTERN etc.), only plain text.
    
    ':': Used to split parameters from their value

    @: References a variable. Can be used after : or without parameter. In this case, the variable referenced must be a list of parameter:value. See the examples above.
       A variable can refer to another variable but cannot refer to itself.

    a-b: Means from scale level 'a' to scale level 'b'. For a single scale level, write 'a' only. Scale levels can be followed by ':' as well as '{' (but never both at once).                 

    ##: Single line comment (is outputted in the resulting mapfile). Cannot be on the same line as uncommented text.    

    //: Single line comment (is not outputted in the resulting mapfile) 

    /* */: Multiline comment (is not outputted in the resulting mapfile)


    Notes
    -----
    1) You should use line jump properly as the current version of the application doesn't support blocks written on a single line
       STYLE {
           COLOR: 255 255 255         -> 'OK'
       } 

       STYLE
       {
           COLOR: 255 255 255         -> 'OK'
       }

       STYLE {COLOR: 255 255 255}     -> 'Wrong'
    
    2) Single line comment of type ## cannot be on the same line as uncommented text.
       ##COLOR: 255 255 255              -> 'OK'

       COLOR: 255 255 255 ##My comment   -> 'Wrong'

    3) In the current version, multiline comment tags /* and */ should be placed at the beginning and at the end of a line or you might get unexpected results.
       /*Multiline   -> 'OK'
       comment*/

       /*
       Multiline     -> 'OK'
       comment
       */

       Mul/*tiline   -> 'Wrong'
       comm*/ent     

Author
------

    Charles-Éric Bourget
    cbourget@mapgears.com

    Bugs report: cbourget@mapgears.com

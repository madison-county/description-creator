# description-creator
Updated Version (V6) / Instructions / Code to create descriptions from polygons 3/20/2023

This process is intended to save money while still following MCA and ARM. Specifically:  https://rules.mt.gov/gateway/RuleNo.asp?RN=24%2E183%2E1110 and https://rules.mt.gov/gateway/ruleno.asp?RN=24%2E183%2E1111

Notes: 
You will have to adjust the variables in the code (step 2) below for your feature class names and for the appropriate output paths. See Bolded text in code. 

The code snippets need to be copied into the Python Pane (for this version). 

I would recommend creating singular districts with this process and then merge the completed datasets together to have one district description with multiple districts within (Commissioner Districts). Alternatively, use your working layers “Definition Query” (as I do in the videos).

The results of this process are intended to be certified by a RLS.

There is a poor quality video of the workflow at:
 	https://www.youtube.com/watch?v=g9ra6T56Zi4
The video missed some key steps but does display the workflow (early stages of dev.)

Process:
1)	Create Polygons for the desired areas. Alternatively, you can create lines directly.

2)	Create variables:		****UPDATE VARIABLES FOR NEW DISTRICTS****

polygon_fc = "FinalDraftPrecincts" # Only needed if converting polygons to lines.
fc = "All_2023_Precincts_Description" 
output_path = r"C:\Users\tluksha\Desktop\Creating_Precincts\Creating_Precincts.gdb"
output_table = "Complete_Descriptions"
ExcelFileName = "All_Precincts.xls"
DistrictName = '"2023 Precinct x-NAME"'
import os  # Do not change this line. It is not a variable.

3)	Convert polygons to lines: (Skip if working with a line FC already)
a.	NOTE: This outputs a feature class titled “Scratch_PolygonToLine”

arcpy.management.PolygonToLine(    in_features= polygon_fc,    out_feature_class= output_path + “Scratch_PolygonToLine",    neighbor_option="IDENTIFY_NEIGHBORS")

4)	Separate (split) the lines where any deviation to the description could occur. 

5)	Join (merge) the lines where they should be continuous but are not. 

6)	Clear Selection from fc: 
a.	Best practice would be to clear within the “Selection” group, In the “Edit” or “Map” tabs versus via code.

arcpy.management.SelectLayerByAttribute(fc, 'CLEAR_SELECTION')

7)	Enter default values: 
arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Area",    expression='"Thence,"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")
arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Approx",    expression='"approximately"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Qualifier",    expression='"miles"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Along",    expression='"along the"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Factors",    expression='"section lines"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="ToThe",    expression='"to the"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Terminus",    expression='"Northeast corner"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="OfSection",    expression='"of section"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table=fc,    field="T",    expression='"T"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Township_Dir",    expression='"S"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table=fc,    field="R",    expression='"R"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table="All_2023_Precincts_Description",    field="Range_Dir",    expression='"W"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table=fc,    field="PMM",    expression='"PMM"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")


8)	Update “District Name” field: (ONLY if working on one district at a time) 

arcpy.management.CalculateField(fc, "District_Name", DistrictName, "PYTHON3", "", "TEXT",  "NO_ENFORCE_DOMAINS")

9)	If the lines are in appropriate order and NOT NUMBERED, run the following code to insert numbers: 

arcpy.management.CalculateField(    in_table="fc",    field="ListOrder",    expression="autoIncrement()",    expression_type="PYTHON3",    code_block="""rec=0def autoIncrement():    global rec    pStart = 3    pInterval = 1    if (rec == 0):        rec = pStart    else:        rec = rec + pInterval    return rec""",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

a.	Manually set “List Order” (if not in order) if needed. 
b.	Change the numbers accordingly after the autoincrement() code has been ran.

10)	Calculate “Course” field: 
arcpy.management.CalculateField(    in_table=fc,    field="Course_Num",    expression='"(Course " + str(!ListOrder!) + ")"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

a.	Alternate (to include master list number): 
arcpy.management.CalculateField(    in_table=fc,    field="Course_Num",    expression='"(Course " + str(!MasterListOrder!) + "-" + str(!ListOrder!) + ")"',    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

11)	Calculate “Bearing”
arcpy.AddGeometryAttributes_management(fc, Geometry_Properties="LINE_BEARING", Length_Unit="", Area_Unit="", Coordinate_System="PROJCS['NAD_1983_StatePlane_Montana_FIPS_2500',GEOGCS['GCS_North_American_1983',DATUM['D_North_American_1983',SPHEROID['GRS_1980',6378137.0,298.257222101]],PRIMEM['Greenwich',0.0],UNIT['Degree',0.0174532925199433]],PROJECTION['Lambert_Conformal_Conic'],PARAMETER['False_Easting',600000.0],PARAMETER['False_Northing',0.0],PARAMETER['Central_Meridian',-109.5],PARAMETER['Standard_Parallel_1',45.0],PARAMETER['Standard_Parallel_2',49.0],PARAMETER['Latitude_Of_Origin',44.25],UNIT['Meter',1.0]]")

12)	Calculate Direction (Indentation issues have occurred from MS Word. Specifically with spacing in front of the “if” statement.		 {See asterisk:    *if (Dir <….}
a.	If Error occurs, Run code from the text file as MS Word continues to crash this (“Calculate Direction from Bearing .txt”). 

arcpy.management.CalculateField(in_table=fc, field="Direction", expression="Reclass(!BEARING!)", expression_type="PYTHON3", code_block="""
def Reclass(Dir):
   if (Dir   < 360 and Dir > 348.75 ): 
       return "North" 
   elif (Dir  > 0 and Dir < 11.25 ):
       return "North"
   elif (Dir  > 11.25 and Dir < 33.75 ): 
       return "NNE"
   elif (Dir  > 33.75 and Dir < 56.25 ):
       return "Northeast"
   elif (Dir  > 56.25 and Dir < 78.75 ):
       return "ENE"
   elif (Dir  > 78.75 and Dir < 101.25 ):
       return "East"
   elif (Dir  > 101.25 and Dir < 123.75 ):
       return "ESE"
   elif (Dir  > 123.75 and Dir < 146.25 ):
       return "Southeast"
   elif (Dir  > 146.25 and Dir < 168.75 ):
       return "SSE"
   elif (Dir  > 168.75 and Dir < 191.25 ):
       return "South"
   elif (Dir  > 191.25 and Dir < 213.75 ):
       return "SSW"
   elif (Dir  > 213.75 and Dir < 236.25 ):
       return "Southwest"
   elif (Dir  > 236.25 and Dir < 258.75 ):
       return "WSW"
   elif (Dir  > 258.75 and Dir < 281.25 ):
       return "West"
   elif (Dir  > 281.25 and Dir < 303.75 ):
       return "WNW"
   elif (Dir  > 303.75 and Dir < 326.25 ):
       return "Northwest"
   elif (Dir  > 326.25 and Dir < 348.75 ):
       return "NNW"
   else:
       return "Error" """,    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

13)	Calculate Distance:
arcpy.management.CalculateGeometryAttributes(    in_features=fc,    geometry_property="DistRound LENGTH",    length_unit="MILES_INT",    area_unit="",    coordinate_system='PROJCS["NAD_1983_StatePlane_Montana_FIPS_2500",GEOGCS["GCS_North_American_1983",DATUM["D_North_American_1983",SPHEROID["GRS_1980",6378137.0,298.257222101]],PRIMEM["Greenwich",0.0],UNIT["Degree",0.0174532925199433]],PROJECTION["Lambert_Conformal_Conic"],PARAMETER["False_Easting",600000.0],PARAMETER["False_Northing",0.0],PARAMETER["Central_Meridian",-109.5],PARAMETER["Standard_Parallel_1",45.0],PARAMETER["Standard_Parallel_2",49.0],PARAMETER["Latitude_Of_Origin",44.25],UNIT["Meter",1.0]]',    coordinate_format="SAME_AS_INPUT")

14)	Round Distance: NOTE: If error occurs, rerun previous step.
arcpy.management.CalculateField(    in_table=fc,    field="DistRound",    expression="round(!DistRound!,2)",    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

15)	Consider code to remove zeros when rounding presents X.0 or X.00

16)	Mile vs Miles
#Update the Qualifier field with mile or miles
arcpy.management.CalculateField(in_table=fc, field="Qualifier", expression="qualifierMiles(!DistRound!)", expression_type="PYTHON3", code_block="""
def qualifierMiles(miles):
    if miles == 1:
        return 'mile'
    else:
        return 'miles'
""",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")


17)	Manually complete or modify the wording for the following fields:
a.	Area			-Generally beginning or ending components.
b.	Factors 		-Modify if needed.
c.	ToThe
d.	Terminus		-DOUBLE CHECK EACH ONE
e.	Section Number	-Numerical only
f.	Township		-Numerical only
g.	Range			-Numerical only
h.	District Name		-Field Calculator is your friend here. 

18)	Join Concatenation (Step 1a):
arcpy.management.CalculateField(    in_table=fc,    field="TR_concat",    expression="\'\'.join(str(i) for i in [!T!, !Township!, !Township_Dir!, \", \", !R!, !Range!, !Range_Dir!, \", \" !PMM!, \"; \"] if i)",    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

arcpy.management.CalculateField(    in_table=fc,    field="Concatonated",    expression="' '.join(str(i) for i in [!Course_Num!, !Area!, !Direction!, !Approx!, !DistRound!, !Qualifier!, !Along!, !Factors!, !LotNum!, !Subdivision!, !ToThe!, !Terminus!, !OfSection!, !SectionNum!,  \",\", !Clarification!, !TR_concat!] if i)",    expression_type="PYTHON3",    code_block="",    field_type="TEXT",    enforce_domains="NO_ENFORCE_DOMAINS")

19)	Table to table (Stripping fields), Then to Excel:
a.	The first line of the following code DELETES the previous table (if one exists with the table name set in the variable section above.
b.	Alternatively, you could rename “output_table”. (I add a numerical if I want to do this)

arcpy.management.Delete(output_table)
arcpy.conversion.TableToTable(    in_rows=fc,    out_path= output_path,    out_name=output_table,    where_clause="",    field_mapping='District_Name "District Name" true true false 300 Text 0 0,First,#,LegalLineCret,District_Name,0,300;Concatonated "Concatonated" true true false 2000 Text 0 0,First,#,LegalLineCret,Concatonated,0,2000',    config_keyword="")

#arcpy.management.Delete(ExcelFileName_Path)

import os
output_path_parent = os.path.dirname(output_path)
ExcelFileName_Path = output_path_parent + """\\""" + ExcelFileName
try:
    os.remove(ExcelFileName_Path)
except:
    pass
arcpy.conversion.TableToExcel(
    Input_Table= output_table,
    Output_Excel_File=ExcelFileName_Path,
    Use_field_alias_as_column_header="ALIAS",
    Use_domain_and_subtype_description="CODE"
)
print(ExcelFileName_Path)
os.startfile(ExcelFileName_Path) #This line will open the Excel file


20)	Hip Hip Hooray??? 




21)	Consider using/creating topology rules.

22)	Turn your lines into Polygons: 
arcpy.management.FeatureToPolygon(    in_features=fc,    out_feature_class=r"C:\Users\tluksha\Desktop\Creating_Precincts\Creating_Precincts.gdb\DRAFT_Precinct_to_Polygon",    cluster_tolerance=None,    attributes="ATTRIBUTES",    label_features=None)
a.	Create code to add fields to output as the “Preserve Attributes” option is no longer supported. 


23)	Set Feature Class variable (fc = fc) before importing code blocks.

24)	Set path variable (output_path=r"C:\Users\tluksha\Desktop\Creating_Commissioner_Legal_Descriptions\Legal_Lines_Creation.gdb") before importing code blocks.

25)	Future revisions should include a script tool or a major code block to copy/paste into the Python pane. 

26)	This could be improved by automating the exports to a final feature class, tables, excel, and word document. All within a completed folder and FGDB for the project.

27)	I hope to find a way to populate a field based upon the end of a line segment. This would allow automation of the Township and Range values.


UPDATE “TR Concatonated” to allow 25 Characters as compared to the 15 that is currently allowed. 

# The Embedded Builder

## FOCUS
The focus of the Embedded Builder is embedded development, it is not a focus of this tool to support Linux or Windows type development.

### That said ....
Yes, plan is to support building test like applications for UNIX and/or windows like environments, but that is only to support some type of test application.

# Definitions

## Simple Ascii Text Name
Means, an ascii text sequence that would serve as a C language variable name.
ie: It starts with a letter or underline, and must consit of exactly letters numbers and underlines

In some cases, simple ascii text names (such as sub-module-names) are required to be unique.  Example:  No Project shall have two modules named: "FOO" the project can have "FOO1" and "FOO2" but not "FOO" and "FOO".

## PROJ_ROOT_DIR and PROJ_ROOT_FILE

The **PROJ_ROOT_DIR** is the filesystem location where the project lives

The only file that Is present in the ROOT_PROJ_DIR
* The **PROJ_ROOT_FILE** is located in the PROJ_ROOT_DIR
* This contains references  (a list) to various SUB_PROJECTs 

Within the PROJ_ROOT_DIR there are SUB_PROJ_DIR locations
There is at least 1 CONFIGURATION SUB_PROJ_DIR within the PROJ_ROOT_DIR
If there is only 1, it should be named Debug
Often there are two, a Debug and Release folder.

There can be many additional SUB_PROJ_DIR folders within the ROOT_PROJ_DIR
Or there maybe references to SUB_PROJ_DIRs that are elsewhere in the file system.
These locations may be absolute paths, or if relative - they are relative to the ROOT_PROJ_DIR.

## SUB_PROJ_DIR and SUB_PROJ_FILE
Each sub project shall have a unique simple ascii text name.

A SUB_PROJECT may be located locally within the ROOT_PROJ_DIR, or it maybe remote
Remote means: Stored elsewhere within the filesystem.

If the SUB_PROJECT is stored locally, the sub project shall have a unique directory name.

The ROOT_PROJECT_FILE can reference a directory stored elsewhere in the filesystem via either (1) an absolute or (2) path relative to the ROOT_PROJECT_FILE, but in each case the SUB_PROJECT must have a unique simple ascii name.

## SUB PROJECT Types

A SUB PROJECT may be any one of:

An application, in windows we call this an EXE file
A static library, generally procuces a ".a" file
A Configuration SubProject (special case of a SUB PROJECT)

## A simple example
 * There are 3 root projects, or ships in your fleet
	* Name:   SAILBOAT,  MOTORBOAT and  ROWBOAT
 * Often SUB_PROJECTS are shared across ROOT_PROJECT
 * Some SUB_PROJECTS are unique to the project it self
 * Example:
	* All boats use the HULL model, just configured differently
	* The motor boat uses the MOTOR module
	* The Sailboat optionally may or may not have a MOTOR module
	* The SAIL boat is the only one with the SAIL module
	* The ROW BOAT is the only one with OARS
	* All boats have various configurations of SAFTY_EQUIPMENT
 * One would choose a configuration and build that configuration (See below, Building)
 
## How do Configurations work?
### Configuration Steps via the GUI
**is this required?** Answer: NO, but please continue reading. 

The goal is this: We would like something like the linux kernel MenuConfig, however we also want to fully support Linux, Mac and Windows development, thus  "kubyz menuconfig" tool is a challange to avoid.  Besides, we have a fully function Python3 already present!

There is no specific requirement for the sub module to produce or provide an HTTP like interface, consider this interface as the "most highly desired and encouraged solution"

Optimmally each SUB_PROJECT should produce or provide a simple web interface to allow for configuration of module. This should be created via the standard python module "simple.http" and not require any special modules.

### Alternatives 
* Provide a .H file that the human manually modifies configures.

### Configuration Output & Result
 * The user selects a named configuration (directory) in the PROJ_ROOT_DIR, under that directory are several sub directories, any and all generated C & H files shall be placed there. 
 * Examples:
 * ${PROJ_ROOT_DIR}/cfg_Debug/include <- All .H files go here, further sub directories are ok here.
 * ${PROJ_ROOT_DIR}/cfg_Debug/src <- all C and C++ files go here
 * ${PROJ_ROOT_DIR}/cfg_Debug/scripts <- script like things go here
 * ${PROJ_ROOT_DIR}/cfg_Debug/build <-- all objects and such go here.
 * 

### JSON Input/Output Requirements 
The resulting .JSON file shall be nested at least one level deep with the module name as the outer most key/tag. The goal is that all configuration items can be stored in a single JSON file and passed to all modules such that then can if they need to - observe other modules configuration needs & requirements, example the modules FOO, CAT, and BAR can be configured via the single JSON object
       JSON_OBJECT = { 
           "FOO" : { ... things for foo go here... },
           "BAR" : { ... things for bar go here ... }
           "CAT" : { ... cat configuration goes here ... }
      }

### When configuration happens, ie: CI/CD mode of operation

The configuration.JSON file will be stored in the file:

    ${ROOT_PROJ_DIR}/cfg_Debug/config.json

When configuration is performed, each module will be called passing (1) the module name as used in the project, and (2) the JSON dict, the module can then extract what it needs from the JSON file

### Building

All building is done within the selected CONFIG_DIR.

Note: When the statement is: "For Each Sub Project" that means:
  Itterate over the subprojects
  The very first is the Configuration project.
  Then each sub project listed in in the ROOT_PROJ_FILE with a priority order
  Priority orders are simple integers and must be unqiue.

### What are project files?

Project files are python files with a single class called SubProject.
One can think of it like this:

Various options work like this:

Configuration:

    ConfigDir = determine_config_dirname()
    for each subproject in priority order:
    	import  subproject.Project as ProjectName
    	ProjectName.configure( config=jsondictionary, configdir=ConfigDir )

Build:

    for each step in ( "prebuild", "build", "postbuild" ):
        for each subproject in priority orderE:
    		import subproject.Project as ProjecName
		ProjectName.Build( configdir=ConfigDir, step =step )

	




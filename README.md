rwflowpack
=========

A role for configuring and managing the rwflowpack service.  rwflowpack is a daemon that runs as part of the SiLK flow collection and packing tool-chain. The primary job of rwflowpack is to convert each incoming flow record to the SiLK Flow format, categorize each incoming flow record (e.g., as incoming or outgoing), set the sensor value for the record, and determine which hourly file will ultimately store the record. See rwflowpack [documentation](https://tools.netsa.cert.org/silk/rwflowpack.html) for more information.

Role Variables
--------------

Available variables are listed below, along with default values (see [defaults/main.yml](defaults/main.yml)):

	silk_packing_tools_loc: "/usr/local/sbin"

The folder location of the silk packing tools.

	rwflowpack_myname: "rwflowpack"

The name of the rwflowpack process.  It is possible to run multiple copies of rwflowpack on a single node by naming them differently.

    rwflowpack_conf_template: "rwflowpack.conf.j2"
    rwflowpack_conf_file_loc: "/usr/local/etc"
    rwflowpack_conf_file_path: "{{ rwflowpack_conf_file_loc }}/{{ rwflowpack_myname }}.conf"
    rwflowpack_init_template: "rwflowpack.j2"
    rwflowpack_init_file_path: "/etc/init.d/{{ rwflowpack_myname }}"

Template sources to use and their destinations.

| Variable  | Explanation |
| ------------- | ------------- |
| rwflowpack_service: True | Whether to enable to the rwflowpack service |
| rwflowpack_statedirectory: "/usr/local/var/lib/rwflowpack" | Directory for where rwflowpack keeps its state |
| rwflowpack_create_directories: "no" | If set to "yes", the defined directories will be created automatically if they do not already exist |
| rwflowpack_bin_dir: "{{ silk_packing_tools_loc }}" | Full path of the directory containing the "rwflowpack" program |
|  data_rootdir: "/data/" | The full path to the root of the tree under which the packed SiLK Flow files will be written.  Used by --root-directory. |
| rwflowpack_packing_logic: "" | Specify the path to the packing-logic plug-in that rwflowpack should load and use.  The plug-in provides functions that determine into which class and type each flow record will be categorized and the format of the files that rwflowpack will write.  When SiLK has been configured with hard-coded packing logic (i.e., when --enable-packing-logic was specified to the configure script), this value should be empty.  A default value for this switch may be specified in the ${SITE_CONFIG} site configuration file.  This value is ignored when INPUT_MODE is "respool". |
| rwflowpack_input_mode: "stream" | Data input mode.  Valid values are: "stream" mode to read from the network or from probes that have poll-directories, "fcfiles" to process flowcap files on the local disk, "respool" to process SiLK flow files maintaining the sensor and class/type values that already exist on those records. |
| rwflowpack_incoming_dir: "" | Directory in which to look for incoming flowcap files in "fcfiles" mode or for incoming SiLK files in "respool" mode |
| rwflowpack_archive_dir: "" | Directory to move input files to after successful processing.  When in "stream" mode, these are the files passed to any probe with a poll-directory directive.  When in "fcfiles" mode, these are the flowcap files.  When in "respool" mode, these are the SiLK Flow files.  If not set, the input files are not archived but are deleted instead. |
| flat_archive: "0" | When using the ARCHIVE_DIR, normally files are stored in subdirectories of the ARCHIVE_DIR.  If this variable's value is 1, files are stored in ARCHIVE_DIR itself, not in subdirectories of it. |
| rwflowpack_error_dir: "" | Directory to move an input file into if there is a problem opening the file.  If this value is not set, rwflowpack will exit when it encounters a problem file.  When in "fcfiles" mode, these are the flowcap files.  When in "stream" mode, these are the files passed to any probe with a poll-directory directive.|
| rwflowpack_output_mode: "local-storage" | Data output mode.  As of SiLK-3.6.0, valid values are "local-storage", "incremental-files", and "sending". In "local-storage" (aka "local") mode, rwflowpack writes the records to hourly files in the repository on the local disk.  The root of the repository must be specified by the DATA_ROOTDIR variable. In "incremental-files" mode, rwflowpack creates small files (called incremental files) that must be processed by rwflowappend to create the hourly files.  The incremental-files are created and stored in a single directory named by the INCREMENTAL_DIR variable. In "sending" (aka "remote") mode, rwflowpack also creates incremental files.  The files are created in directory specified by the INCREMENTAL_DIR variable and then moved to directory specified by the SENDER_DIR variable. |
| rwflowpack_sender_dir: "{{ rwflowpack_statedirectory }}/sender-incoming" | When the OUTPUT_MODE is "sending", this is the destination directory in which the incremental files are finally stored to await processing by rwflowappend, rwsender, or another process. |
| rwflowpack_incremental_dir: "{{ rwflowpack_statedirectory }}/sender-incoming" | When OUTPUT_MODE is "incremental-files" or "sending", this is the directory where the incremental files are initially built.  In "incremental-files" mode, the files remain in this directory.  In "sending" mode, the incremental files are moved to the SENDER_DIR directory. |
| rwflowpack_compression_type: "" | The type of compression to use for packed files.  Left empty, the value chosen at compilation time will be used.  Valid values are "best" and "none".  Other values are system-specific (the available values are listed in the description of the --compression-method switch in the output of rwflowpack --help). |
| rwflowpack_polling_interval: "15" | Interval between attempts to check the INCOMING_DIR or poll-directory probe entries for new files, in seconds.  This may be left blank, and will default to 15. |
| rwflowpack_flush_timeout: "120" | Interval between periodic flushes of open SiLK Flow files to disk, in seconds.  This may be left blank, and will default to 120. |
| rwflowpack_cache_size: "64" | Maximum number of SiLK Flow files to have open for writing simultaneously.  This may be left blank, and will default to 64 |
| rwflowpack_file_locking: "1" | Whether rwflowpack should use advisory write locks.  1=yes, 0=no. Set to zero if messages like "Cannot get a write lock on file" appear in rwflowpack's log file. |
| rwflowpack_pack_interfaces: "0" | Whether rwflowpack should include the input and output SNMP interfaces and the next-hop-ip in the output files.  1=yes, 0=no. The default is no, and these values are not stored to save disk space.  (The input and output fields contain VLAN tags when the sensor.conf file contains the attribute "interface-values vlan".) |
| rwflowpack_silk_ipfix_print_templates: "" | Setting this environment variable to 1 causes rwflowpack to log the NetFlowV9/IPFIX templates that it receives. |
| rwflowpack_log_type: "syslog" | The type of logging to use.  Valid values are "legacy" and "syslog". |
| rwflowpack_log_level: "info" | The lowest level of logging to actually log.  Valid values are: emerg, alert, crit, err, warning, notice, info, debug |
| rwflowpack_log_dir: "{{ rwflowpack_statedirectory }}/log" | The full path of the directory where the log files will be written when LOG_TYPE is "legacy". |
| rwflowpack_pid_dir: "{{ rwflowpack_log_dir }}" | The full path of the directory where the PID file will be written |
| rwflowpack_user: "root" | The user this program runs as; root permission is required only when rwflowpack listens on a privileged port. |
| rwflowpack_extra_options: "" | Extra options to pass to rwflowpack |
| rwflowpack_extra_envvar: "" | Extra environment variables to set when running rwflowpack. |

As part of collecting flow data, the rwflowpack and flowcap daemons need to know what type of data they are collecting and how to collect it (e.g., listen on 10000/udp for NetFlow v5; listen on 4740/tcp for IPFIX). In addition, the rwflowpack daemon needs information on how to categorize the flow: for example, to label the flows collected at a border router as incoming or outgoing. The Sensor Configuration file, sensor.conf, contains this information, and the following variables control it's location and contents.  See the [manual page](https://tools.netsa.cert.org/silk/sensor.conf.html) for descriptions of the syntax of the file (see [SYNTAX](https://tools.netsa.cert.org/silk/sensor.conf.html#SYNTAX)) and some example configurations (see [EXAMPLES](https://tools.netsa.cert.org/silk/sensor.conf.html#EXAMPLES)).

| sensor.conf Variable  | Explanation |
| ------------- | ------------- |
| sensor_conf_template: "sensor.conf.j2" | Jinja template source for sensor.conf file |
| sensor_conf_file_loc: "{{ data_rootdir }}" | The directory to copy sensor.conf to |
| sensor_conf_file_path: "{{ sensor_conf_file_loc }}sensor.conf" | The full path to copy sensor.conf to |
| sensor_conf_file_contents: "" | The contents of the sensor.conf file |

The silk.conf SiLK site configuration file is used to associate symbolic names with flow collection information stored in SiLK Flow records.  In addition to the information contained in the NetFlow or IPFIX flow record (e.g., source and destination addresses and ports, IP protocol, time stamps, data volume), every SiLK Flow record has two additional pieces of information that is added when rwflowpack(8) converts the NetFlow or IPFIX record to the SiLK format: the sensor and the flowtype.  See the [manual page](https://tools.netsa.cert.org/silk/silk.conf.html) for more information about the usage and syntax of this file.

| silk.conf Variable  | Explanation |
| ------------- | ------------- |
| silk_conf_template: "silk.conf.j2" | Jinja template source for silk.conf file |
| silk_conf_file_loc: "{{ data_rootdir }}" | The directory to copy silk.conf to |
| silk_conf_file_path: "{{ silk_conf_file_loc }}silk.conf" | The full path to copy silk.conf to |
| sensor_definitions: "" | The contents of the sensor block within the silk.conf file.  Individual sensor definitions are created with the sensor command. This command creates a new sensor with the given name and numeric ID. Sensor names must begin with a letter, must not be longer than 64 characters, and may not contain whitespace characters or the characters slash (/) or underscore (_). |
| class_block_definitions: "" | The contents of the class block(s) for mapping which sensor(s) belong to it. |
| class_type_definitions | The contents of the class's type.  By default this role assumes you will be using twoway packing logic |
| default_class: "all" | rwfilter and rwfglob will use a default class when the user does not specify an explicit --class. This command specifies that default class; the class must have been created prior to this command. If more than one default class is set, the last definition encountered is used. |
| packing_logic_plugin: "packlogic-twoway.so" | The packing-logic command provides a default value for the --packing-logic switch on rwflowpack. The value is the path to a plug-in that rwflowpack loads; the plug-in provides functions that determine into which class and type a flow record will be categorized. The path specified here will be ignored when the --packing-logic switch is explicitly specified to rwflowpack or when SiLK has been configured with hard-coded packing logic. |

Dependencies
------------

- cmusei.silk

Example Playbook
----------------

    - hosts: servers
      vars:
        rwflowpack_create_directories: "yes"
        sensor_definitions: |
            sensor 0 S0    "Description for sensor S0"
        class_block_definitions: |
            class all
                sensors S0
            end class
        sensor_conf_file_contents: |
            probe S0 ipfix
                poll-directory /tmp
            end probe
            
            sensor S0
                ipfix-probes S0
                internal-ipblock 128.2.0.0/16
                external-ipblock remainder
            end sensor
      roles:
        - role: cmusei.rwflowpack
          tags: [ 'rwflowpack' ]

License
-------

Copyright 2020 Carnegie Mellon University.
NO WARRANTY. THIS CARNEGIE MELLON UNIVERSITY AND SOFTWARE ENGINEERING INSTITUTE MATERIAL IS FURNISHED ON AN "AS-IS" BASIS. CARNEGIE MELLON UNIVERSITY MAKES NO WARRANTIES OF ANY KIND, EITHER EXPRESSED OR IMPLIED, AS TO ANY MATTER INCLUDING, BUT NOT LIMITED TO, WARRANTY OF FITNESS FOR PURPOSE OR MERCHANTABILITY, EXCLUSIVITY, OR RESULTS OBTAINED FROM USE OF THE MATERIAL. CARNEGIE MELLON UNIVERSITY DOES NOT MAKE ANY WARRANTY OF ANY KIND WITH RESPECT TO FREEDOM FROM PATENT, TRADEMARK, OR COPYRIGHT INFRINGEMENT.
Released under a MIT (SEI)-style license, please see license.txt or contact permission@sei.cmu.edu for full terms.
[DISTRIBUTION STATEMENT A] This material has been approved for public release and unlimited distribution.  Please see Copyright notice for non-US Government use and distribution.
CERT® is registered in the U.S. Patent and Trademark Office by Carnegie Mellon University.
This Software includes and/or makes use of the following Third-Party Software subject to its own license:
1. ansible (https://github.com/ansible/ansible/tree/devel/licenses) Copyright 2019 Red Hat, Inc.
2. molecule (https://github.com/ansible-community/molecule/blob/master/LICENSE) Copyright 2018 Red Hat, Inc.
3. testinfra (https://github.com/philpep/testinfra/blob/master/LICENSE) Copyright 2020 Philippe Pepiot.

DM20-0532


Author Information
------------------

This role was created in 2020 by [Matt Heckathorn](https://resources.sei.cmu.edu/library/author.cfm?authorID=2403).

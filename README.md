# Sample-Docker-File-creation-with-bash-script
The script is used to create a base template docker file based on file extension as output.
# file_type_segregation
#!/bin/bash

if [ $# -ne 2 ]; then
	echo "Error: Number of arguments not equal to 2"
	echo "Info: Expecting 2 arguments, arg1 = project working directory, arg2 = Docker file template directory"
	exit 1
fi

dir_name="$1"
docker_temp_dir=$2
extn="$2"
search_extn="*.""$extn"

segregate_files() {

	find "$dir_name" -type f | sed -n 's/..*\.//p' | sort | uniq -c > seg_file.txt
	echo "Details of file extensions inside directory "$dir_name" and corresponding counts are written to file "seg_file.txt""
}

find_files() {
	find "$dir_name" -type f -name "$search_extn" > file_details.txt
	echo "Details of files with extension "$search_extn" inside "$dir_name" are written to file "file_details.txt""
}


if [ -d "$dir_name" ]; then

	segregate_files
	find_files
	if [ $? -eq 0 ]; then
		sh ./create_docker_file.sh $dir_name $docker_temp_dir
	fi
else
	echo "Error: Directory $dir_name does not exists!!"
	exit 1
fi

# Create docker file
#!/bin/bash

if [ $# -ne 2 ]; then
	echo "Error: Number of arguments not equal to 2"
	echo "Info: Expecting 1 argument, arg1 = project working directory, arg2 = Docker file template directory"
	exit 1
fi

extn_details=seg_file.txt
work_dir=$1
dock_temp_dir=$2
python_template=docker_python
java_template=docker_java
python_props=python_apps.properties
java_props=java_apps.properties


check_extension() {
	if [ $2 = "python" ]; then
		echo "Building Docker File for "  $2
		cp $dock_temp_dir/$python_template $work_dir
		sh ./read_props.sh $work_dir $python_props $python_template
	elif [ $2 = "java" ]; then
		echo "Building Docker File for" $2
		cp $dock_temp_dir/$java_template $work_dir
		sh ./read_props.sh $work_dir $java_props $java_template
	elif [ $2 = "net" ]; then
		echo $2
	fi
        
}


while IFS='' read -r line; do
	#echo "Text read from file: $line"	
	check_extension $line
done < $extn_details

# create properties

#!/bin/bash
if [ $# -ne 3 ]; then
	echo "Error: Number of arguments not equal to 3"
	#echo "Info: Expecting 1 arguments, arg1 = application properties path"
	exit 1
fi
WORK_DIR=$1
PROP_FILE=$2
DOCK_TEMP=$3

while IFS='=' read -r k v; do
	echo "Replacing variable in docker template" $k " With value " $v
	sed -i "s/$k/$v/g" $WORK_DIR/$DOCK_TEMP
done < $WORK_DIR/$PROP_FILE
echo "Docker File created Successfully and placed at " $WORK_DIR "and Docker File Name is:" $DOCK_TEMP

pipeline {
  agent any
  parameters {
    string(name: 'IMAGE_NAME', defaultValue: 'clever_noname.img', description: 'Output image file name')
    string(name: 'GWBT_REF', defaultValue: 'master', description: 'Checkout ref-param')
    string(name: 'IMAGE_VERSION', defaultValue: 'no_version', description: 'Image version')

    string(name: 'BUILD_DIR', defaultValue: '/mnt/hdd_builder/workspace', description: 'Build workspace')

    string(name: 'RPI_DONWLOAD_URL', defaultValue: 'http://director.downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2018-06-29/2018-06-27-raspbian-stretch-lite.zip')
    // TODO: Add mirrorparameters

    string(name: 'GWBT_URL', defaultValue: 'https://github.com/CopterExpress/clever.git')

    // Experimental function
    booleanParam(name: 'SHRINK', defaultValue: true, description: 'SHRINK IMAGE')
    booleanParam(name: 'DISCOVER_ROS_PACKAGES', defaultValue: false, description: 'DISCOVER ROS PACKAGES')
  }
  environment {
    DEBIAN_FRONTEND = 'noninteractive'
    LANG = 'C.UTF-8'
    LC_ALL = 'C.UTF-8'
  }
  stages {
    stage('Get image') {
      steps {
        sh "$WORKSPACE/image_builder/image_config.sh get_image ${params.BUILD_DIR} ${params.RPI_DONWLOAD_URL} ${params.IMAGE_NAME}"
      }
    }
    stage('Resize FS') {
      environment {
        SIZE = '7G'
      }
      steps {
        sh "$WORKSPACE/image_builder/image_config.sh resize_fs ${params.BUILD_DIR}/${params.IMAGE_NAME} $SIZE"
      }
    }
    stage('Initialize image') {
      environment {
        EXECUTE_FILE = 'image_builder/scripts/init_image.sh'
      }
      // TODO: Transfer apps.sh initialisation code here
      steps {
        sh "$WORKSPACE/image_builder/image_config.sh execute ${params.BUILD_DIR}/${params.IMAGE_NAME} $WORKSPACE/$EXECUTE_FILE ${params.IMAGE_VERSION} \$(basename ${params.RPI_DONWLOAD_URL})"
      }
    }
    stage('Hardware setup') {
      environment {
        EXECUTE_FILE = 'image_builder/scripts/hardware_setup.sh'
      }
      steps {
        sh "$WORKSPACE/image_builder/image_config.sh execute ${params.BUILD_DIR}/${params.IMAGE_NAME} $WORKSPACE/$EXECUTE_FILE"
      }
    }
    stage('Software install') {
      environment {
        EXECUTE_FILE = 'image_builder/scripts/software_install.sh'
      }
      steps {
        sh "$WORKSPACE/image_builder/image_config.sh execute ${params.BUILD_DIR}/${params.IMAGE_NAME} $WORKSPACE/$EXECUTE_FILE"
      }
    }
    stage('Network setup') {
      environment {
        EXECUTE_FILE = 'image_builder/scripts/network_setup.sh'
      }
      steps {
        sh "$WORKSPACE/image_builder/image_config.sh execute ${params.BUILD_DIR}/${params.IMAGE_NAME} $WORKSPACE/$EXECUTE_FILE"
      }
    }
    stage('Install ROS') {
      environment {
        EXECUTE_FILE = 'image_builder/scripts/ros_install.sh'
        MOVE_FILE = 'image_builder/kinetic-ros-coex.rosinstall'
        MOVE_TO = '/home/pi/ros_catkin_ws'
      }
      steps {
        sh "if ! ${params.DISCOVER_ROS_PACKAGES}; then $WORKSPACE/image_builder/image_config.sh copy_to_chroot ${params.BUILD_DIR}/${params.IMAGE_NAME} $WORKSPACE/$MOVE_FILE $MOVE_TO; fi"
        sh "$WORKSPACE/image_builder/image_config.sh execute ${params.BUILD_DIR}/${params.IMAGE_NAME} $WORKSPACE/$EXECUTE_FILE ${params.GWBT_URL} ${params.GWBT_REF} ${params.DISCOVER_ROS_PACKAGES}"
      }
    }
    // TODO: Add finalising step, transfer mirror removal from ros.sh
    stage('Shrink image') {
      environment {
        EXECUTE_FILE = 'image_builder/scripts/change_boot_part.sh'
      }
      when { expression { return params.SHRINK } }
      steps {
        sh "$WORKSPACE/image_builder/autosizer.sh ${params.BUILD_DIR}/${params.IMAGE_NAME}"
        sh "$WORKSPACE/image_builder/image_config.sh execute ${params.BUILD_DIR}/${params.IMAGE_NAME} $WORKSPACE/$EXECUTE_FILE"
      }
    }
  }
}

cd ~/ros2_ws/src

ros2 pkg create --build-type ament_cmake mon_interfaces

cd mon_interfaces

mkdir srv

nano srv/Calcul.srv

----colle:

    float64 a
    float64 b
    string op
   
    float64 result
    bool success
---------

nano package.xml

----ajoute avant `<export>`:

    <buildtool_depend>rosidl_default_generators</buildtool_depend>
    <exec_depend>rosidl_default_runtime</exec_depend>
    <member_of_group>rosidl_interface_packages</member_of_group>
------------

nano CMakeLists.txt

----colle:

    find_package(ament_cmake REQUIRED)
    
----au dessous de find_package(rosidl_default_generators REQUIRED)

----et colle:

    rosidl_generate_interfaces(${PROJECT_NAME}
     "srv/Calcul.srv"
    )

----avant ament_package()

------------------

cd ~/ros2_ws/src

ros2 pkg create --build-type ament_python calculatrice_pkg

cd calculatrice_pkg/calculatrice_pkg

nano calculatrice_node.py

----colle dans le fichier:

    from mon_interfaces.srv import Calcul

    import rclpy

    from rclpy.node import Node


    class Calculatrice(Node):

    def __init__(self):

        super().__init__('calculatrice')

        self.srv = self.create_service(
            Calcul,
            '/calcule',
            self.callback
        )

    def callback(self, req, resp):

        ops = {
            '+': req.a + req.b,
            '-': req.a - req.b,
            '*': req.a * req.b
        }

        if req.op == '/' and req.b != 0:
            ops['/'] = req.a / req.b

        resp.result = ops.get(req.op, 0.0)

        resp.success = req.op in ops

        return resp


    def main(args=None):

    rclpy.init(args=args)

    node = Calculatrice()

    rclpy.spin(node)

    rclpy.shutdown()


    if __name__ == '__main__':
      main()
----------
cd ~/ros2_ws/src/calculatrice_pkg

nano.setup.py

----remplace dans:
      
    entry_points={

    'console_scripts': [
    
----avec:

    entry_points={
   
    'console_scripts': [
        'calculatrice_node = calculatrice_pkg.calculatrice_node:main',
    ],
    },
-----------------------

nano package.xml

----ajoute avant test depend:

`<depend>rclpy</depend>`

`<depend>mon_interfaces</depend>`

-------------------

cd ~/ros2_ws

source /opt/ros/jazzy/setup.bash

colcon build

source install/setup.bash


ros2 run calculatrice_pkg calculatrice_node

----on a un terminal vide

-----dans le 2eme terminal:

source /opt/ros/jazzy/setup.bash

source ~/ros2_ws/install/setup.bash

----exmpl 5+2:

ros2 service call /calcule mon_interfaces/srv/Calcul "{a: 5.0, b: 2.0, op: '+'}"

requester: making request: mon_interfaces.srv.Calcul_Request(a=5.0, b=2.0, op='+')

response:

mon_interfaces.srv.Calcul_Response(result=7.0, success=True)

----exmpl 10/2:

ros2 service call /calcule mon_interfaces/srv/Calcul "{a: 10.0, b: 2.0, op: '/'}"

requester: making request: mon_interfaces.srv.Calcul_Request(a=10.0, b=2.0, op='/')

response:

mon_interfaces.srv.Calcul_Response(result=5.0, success=True)





  

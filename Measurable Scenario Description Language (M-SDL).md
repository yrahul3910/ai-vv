# Measurable Scenario Description Language (M-SDL)

* Using M-SDL, you can create scenarios that describe the behavior of the AV as well as other actors in the environment, such as other vehicles, pedestrians, weather, road conditions and so on

* Because M-SDL scenarios are high-level, abstract descriptions, you can create many concrete variants of a scenario by varying the scenario parameters, such as speed, vehicle type, weather conditions and so on

* The Measurable Scenario Description Language (M-SDL) is a mostly declarative programming language. The only scenario that executes automatically is the top-level scenario. You control the execution flow of the program by adding scenarios to the top-level scenario. (**like make**)

* The building blocks of M-SDL are data structures:

    * Simple structs – a basic entity containing attributes, constraints and so on.
    * Actors – like structs, but also have associated scenarios.
    * Scenarios – describe the behavior of actors.

* You can control attribute values with **keep()** constraints, for example:

    * ```
        keep(speed < 50kph)
        ```

* **Example 1** shows how to define and extend an actor. The actor **car_group** is initially defined with two attributes. Then it is extended in a different file to add another attribute.

    * ```
        # Define an actor
        actor car_group:
          average_distance: distance 
          number_of_cars: uint
          
        # Extend an actor in a separate file 
        import car_group.sdl
        
        extend car_group: 
        	average_speed: speed
        ```

* **Example 2** shows how to define a new scenario called **two_phases**. It defines a single attribute, **car1**, which is a **green truck**. It uses the **serial** and **parallel** operators to activate the **car1.drive** scenario, and it applies the **speed()** modifier. During the first phase, **car1** accelerates from 0kph to 10kph. During the second phase, **car1** keeps a speed between 10kph and 15kph.

    ```
    # A two-phase scenario
    scenario traffic.two_phases: # Scenario name
        # Define the cars with specific attributes
        car1: car with:
    				keep(it.color == green) 
    				keep(it.category == truck)
    				
        path: path # a route from the map; specify map in the test
        
        # Define the behavior
        do serial:
    			phase1: car1.drive(path) with: 
    				spd1: speed(0kph, at: start) 
    				spd2: speed(10kph, at: end)
    			phase2: car1.drive(path) with: 
    				speed([10..15]kph)
    ```

* **Example 3** shows how to define the test to be run: 

    * \1. Import the proper configuration. In this case we want to run this test with the SUMO simulator.
    * \2. Import the **two_phases** scenario we defined before.
    * \3. Extend the predefined, initially empty **top.main** scenario to invoke the imported **two_phases** scenario.

    ```
    import sumo_config.sdl 
    import two_phases.sdl
    
    extend top.main:
    	set_map("/maps/hooder.xodr") # specify map to use in this test do two_phases()
    ```

* **Example 4** shows how to define the **cut in** scenario. In it, **car1** cuts in front of the **dut.car**, either from the left or from the right. **dut.car**, also called the **ego** car, is predefined.

    ```
    # The cut-in scenario
    scenario dut.cut_in():
    	car1: car # The other car
    	side: av_side # A side: left or right 
    	path: path
    	
    	path_min_driving_lanes(path, 2) # Needs at least two lanes
    	
    	do serial():
    		get_ahead: parallel(duration: [1..5]s): # get_ahead is a label
    			dut.car.drive(path) with: 
    				speed([30..70]kph)
    			car1.drive(path, adjust: true) with: 
    				position(distance: [5..100]m,
    								behind: dut.car, at: start) 
    				position(distance: [5..15]m,
    								ahead_of: dut.car, at: end)
    								
    	change_lane: parallel(duration: [2..5]s): # change_lane is a label
    		dut.car.drive(path) 
    		car1.drive(path) with:
    			lane(side_of: dut.car, side: side, at: start) 
    			lane(same_as: dut.car, at: end)
    ```

* **Example 5** shows how to define the **two_cut_in** scenario using the **cut_in** scenario. It executes a cut_in from the left followed by a cut_in from the right. Furthermore, the colors of the two cars involved are constrained to be different.

    ```
    # Do two cut-ins serially import cut_in.sdl
    scenario dut.two_cut_ins(): 
    	do serial():
    		c1: cut_in() with: 
    			keep(it.side == left)
    		c2: cut_in() with:
    			keep(it.side == right) 
    			keep(c1.car1.color != c2.car1.color)
    ```


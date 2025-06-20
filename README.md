
EdgeSimPy is a Python-based Edge Computing simulator with easy-to-grasp abstractions for edge servers, network devices, and applications, and built-in models for user mobility, application composition, and power consumption. An overview of EdgeSimPy use cases is shown in the figure below.

<img src="/docs/assets/edgesimpy-features.jpg" alt="EdgeSimPy Use Cases" width="60%" />


EdgeSimPy was designed to streamline the prototyping of resource management policies through realistic simulations. To do this, EdgeSimPy provides a set of unique functional abstractions (e.g., container registries, images, and layers) that replicate the application provisioning method of widely used platforms such as [Docker](https://www.docker.com/)




## Input Files

Before diving into EdgeSimPy, you'll need a scenario input file written in JSON. EdgeSimPy input files must be organized according to a well-defined structure comprised of two distinct information groups: **attributes** and **relationships**.

Attributes refer to the internal characteristics of entities, such as edge server capacity, network link bandwidth, application delay, among others. Relationships represent the associations between entities (e.g., a service's host or a user's applications).

By adhering to this predefined structure, EdgeSimPy can automatically identify entity input metadata and construct the simulated scenario, even in cases where custom attributes and relationships have been specified. A sample dataset file following EdgeSimPy's input format is shown below.

<img src="/docs/assets/edgesimpy-input-format.jpg" alt="EdgeSimPy Input Format" width="50%" />



## Monitoring

Once the simulation starts, EdgeSimPy monitor the entity's state at the end of each time step. Simulation logs are stored in MessagePack, a fast binary serialization format. Instead of writing data to disk each time step, EdgeSimPy stores the simulation output at configurable intervals of time steps, reducing the I/O pressure during the simulation. You can also customize which entity metrics are monitored at each time step by overriding the entity's `collect()` method.

## Components

EdgeSimPy's flexibility stems from a modular architecture, where each entity is self-contained to streamline the integration of new features and algorithms. An overview of EdgeSimPy's architecture is presented in the figure below.

<img src="/docs/assets/edgesimpy-architecture.jpg" alt="EdgeSimPy Architecture" width="60%" />

EdgeSimPy's functional abstractions are grouped into four layers:

**➡️ Core Layer:** Comprises essential libraries and functions for data loading, time progression, and entity monitoring.

**➡️ Physical Layer:** Contains functional abstractions for entities with geospatial information (e.g., users, servers, and network devices).

**➡️ Logical Layer:** Comprises functional abstractions for applications running on the edge infrastructure. It is worth noting that EdgeSimPy adopts containerization as the default virtualization model.

**➡️ Management Layer:** Define the primary resource allocation decisions that can be simulated using EdgeSimPy, which include service placement and migration, maintenance operations, and network flow scheduling.


You can find more details on EdgeSimPy's functional abstractions below:

- **Base Stations:** Act as gateways in the edge network, providing wireless connectivity for seamless communication between users and edge servers. Base stations on EdgeSimPy embody multiple customizable attributes (e.g., energy consumption and wireless latency).
- **Network Switches:** Provide wired connectivity between infrastructure components (e.g., base stations and edge servers) and manage data flows in the network. Network switches ship multiple configurable parameters (e.g., chassis types and varying numbers of ports with specific delay and bandwidth properties). EdgeSimPy models the network topology using using [NetworkX](https://networkx.org/), a well-known graph library for manipulating complex networks that ships several built-in methods (e.g., shortest path and community finding)
- **Edge Servers:** Edge servers are used to host services. Edge servers can ship multiple parameters for capacity (CPU/RAM/disk) and performance (Million Instructions Per Second). EdgeSimPy's power modeling enables the implementation of advanced features, such as temporarily turning off edge servers to save energy. As the properties of power models are fully encapsulated, EdgeSimPy supports custom power models for edge servers (by default, EdgeSimPy incorporates three generic power models: [LinearPowerModel](https://github.com/EdgeSimPy/EdgeSimPy/blob/master/edge_sim_py/components/power_models/servers/linear_server_power_model.py), [QuadraticPowerModel](https://github.com/EdgeSimPy/EdgeSimPy/blob/master/edge_sim_py/components/power_models/servers/square_server_power_model.py), and [CubicPowerModel](https://github.com/EdgeSimPy/EdgeSimPy/blob/master/edge_sim_py/components/power_models/servers/cubic_server_power_model.py)). As edge servers have static coordinates, they are immobile by default. Nevertheless, EdgeSimPy can be extended to assign mobility models to edge servers, allowing the representation of mobile devices with computing capabilities, such as drones or Single-Board Computers (SBCs) connected to automobiles.
- **Users:** Users can either remain in the same position during the entire simulation or move according to mobility models. By default, EdgeSimPy incorporates two mobility models, Random and Pathway, which can be easily replaced by other synthetic models or real mobility traces. Users and applications are linked by a many-to-many relationship, meaning that a user can access multiple applications or even an application to be accessed jointly by multiple users. Users have properties that define their delay and availability requirements for each application they access (we can also add new requirements such as security and budget without burden by leveraging EdgeSimPy's flexible input format). Users also have their access patterns, specifying when they will call their applications and how long each access will last. By default, EdgeSimPy incorporates two user access pattern templates, Random and Circular. While the former arbitrarily defines when and for how long the user will access their applications, the latter establishes a pattern that repeats indefinitely. Based on this, we can use EdgeSimPy to model different workloads, from streaming to batch processing applications and serverless functions.
- **Applications:** Abstract entities representing data flows involving multiple services. This way, the application services are allocated within the infrastructure rather than the applications themselves. As EdgeSimPy models applications as self-contained entities, they can receive custom attributes, such as priority and budget, which enables modeling specific scenarios.
- **Services:** Container instances within the infrastructure. While a service's disk demand corresponds to the size of the layers that comprise its container image, its CPU and memory demand describe the computational resources required by the service instance and therefore are unrelated to the service's image. Each service also has a state attribute, which defines whether it is stateless or stateful.
- **Container Registries:** Containerized services built on top of a registry image that embed image distribution and storage functionality. Container registries are the most important component for service allocations in the edge infrastructure, as service container images are pulled from them to the destination host.
- **Container Images:** Embed the basic functionality for services. Like applications, container images are modeled as abstract entities, so they have no resource requirements by themselves. Instead, the disk demand of a given container image results from the size of its layers.
- **Container Layers:** Represent the instructions aggregated into container images. Each container layer carries attributes representing its software instruction and disk size. As container images in EdgeSimPy adhere to a layered filesystem model, co-hosted services can share read-only image data, resulting in considerable disk savings.




```python
def get_sorted_list_of_edge_servers(service_to_host):
    # The list below will accommodate a dictionary with the edge server objects and their "score" value
    edge_servers = []
    
    # Let's loop through the list of all edge servers, assigning each a sample "score" value. In this
    # case, our score is a randomly generated value, ranging between the sum of the edge server ID and
    # the service ID, all the way up to 1000
    for edge_server in EdgeServer.all():
        edge_server_metadata = {
            "object": edge_server,
            "score": random.randint(edge_server.id + service.id, 1000),
        }
        edge_servers.append(edge_server_metadata)
    
    # Let's sort the list of edge servers according to their score values. Here, we use
    # Python's sorted() function with the reverse attribute as we want to position edge
    # servers with higher score values first in the list
    sorted_list_of_edge_servers_metadata = sorted(edge_servers, key=lambda server_metadata: server_metadata["score"], reverse=True)
    
    # After sorting edge servers, let's get rid of the score values as we will not use them anymore
    sorted_list_of_edge_servers = [server_metadata["object"] for server_metadata in sorted_list_of_edge_servers_metadata]

    return sorted_list_of_edge_servers


def my_algorithm(parameters):
    # We can always call the 'all()' method to get a list with all created instances of a given class
    for service in Service.all():
        # We don't want to migrate services are are already being migrated
        if service.server == None and not service.being_provisioned:

            # We are going to call an external function to get us a sorted list of edge servers
            edge_servers = get_sorted_list_of_edge_servers(service_to_host=service)
            
            # Let's iterate over the list of edge servers to find a suitable host for our service
            for edge_server in edge_servers:
                
                # We must check if the edge server has enough resources to host the service
                if edge_server.has_capacity_to_host(service=service):
                    
                    # Start provisioning the service in the edge server
                    service.provision(target_server=edge_server)
                    
                    # After defining a host server for the service we can move on to the next service
                    break
```



In our sample service placement policy, for each service that we need to provision in the edge infrastructure, an example function is used to rank the list of host candidates. In our example, for the sake of simplicity, the "score" value for each edge server is calculated using a rather arbitrary function. Of course, this placeholder logic can be effortlessly replaced according to our specific research goals.



> Please note that the code above has been written to ease the understanding of researchers who may not be as familiar with Python syntax. Therefore, we have intentionally not used some language optimizations and best practices.



After implementing our service placement policy, we can instantiate an object from EdgeSimPy's "Simulator" class, providing some simulation details such as the stopping criterion (in our case, when all services have been provisioned) and an input dataset file:

```python
# Creating a Simulator object
simulator = Simulator(
    tick_duration=1,
    tick_unit="seconds",
    stopping_criterion=lambda model: all(service.server != None for service in Service.all()),
    resource_management_algorithm=my_algorithm,
)

# Loading a sample dataset and running the simulation
simulator.initialize(input_file="sample_dataset.json")
simulator.run_model()

# Checking the placement output
for service in Service.all():
    print(f"{service}. Host: {service.server}")
```



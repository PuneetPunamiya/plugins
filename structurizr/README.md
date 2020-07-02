# Structurizr Plugin

The `structurizr` plugin is a [Goa](https://github.com/goadesign/goa/tree/v3)
plugin that augments the Goa DSL with keywords that make it possible to
describe a system architecture.

The plugin uses the [C4 model](https://c4model.com) to describe the
architecture and produces JSON that is compatible with the
[Structurizr](https://structurizr.com) service.

## Example:

```Go
var _ = Workspace("Getting Started", "This is a model of my software system.", func() {
    var System = SoftwareSystem("Software System", "My software system.")

    var User = Person("User", "A user of my software system.", func() {
        Uses(System, "Uses")
    })

    Views(func() {
        SystemContext(MySystem, "SystemContext", "An example of a System Context diagram.", func() {
            IncludeAll()
            AutoLayout()
        })
        Styles(func() {
            Element(System, func() { // Element("Software System", ...) also works
                Background("#1168bd")
                Color("#ffffff")
             })
            Element(User, func() { // Element("User", ...) also works
                Shape("Person")
                Background("#08427b")
                Color("#ffffff")
            })
        })
    })
})
```

This code creates a model containing elements and relationships, creates a single view and adds some styling.
![Getting Started Diagram](https://structurizr.com/static/img/getting-started.png)

## Usage

Simply include the plugin DSL package in your design:

```Go
package design

import . "goa.design/goa/v3/dsl"
import . "goa.design/plugins/structurizr/dsl"

// ...
```

Running `goa gen` creates a `structurizr.json` file in the `gen` folder. This
file follows the
[structurizr JSON schema](https://github.com/structurizr/json). 

## Complete syntax:

```Go
// Workspace defines the workspace containing the models and views. Workspace
// must appear exactly once in a given design. A name must be provided if a
// description is.
var _ = Workspace("[name]", "[description]", func() {

    // Enterprise provides a way to define a named "enterprise" (e.g. an
    // organisation). On System Landscape and System Context diagrams, an
    // enterprise is represented as a dashed box. Only a single enterprise
    // can be defined within a model.
    Enterprise("<name>")

    // Person defines a person (user, actor, role or persona).
    var PersonIdentifier = Person("<name>", "[description]", func() { // optional
        Tag("<name>", "[name]") // as many tags as needed

        // URL where more information about this system can be found.
        URL("<url>")

        // External indicates the person is external to the enterprise.
        External()

        // Adds a uni-directional relationship between this person and the given element.
        Uses(ElementIdentifier, "<description>", "[technology]", Synchronous) /* or Asynchronous */)

        // Adds an interaction between this person and another.
        InteractsWith(PersonIdentifier, "<description>", "[technology]", Synchronous) /* or Asynchronous */)
    })

    // SoftwareSystem defines a software system.
    var SoftwareSystemIdentifier = SoftwareSystem("<name>", "[description]", func() { // optional
        Tag("<name>",  "[name]") // as many tags as needed

        // URL where more information about this software system can be
        // found.
        URL("<url>") 

        // External indicates the software system is external to the enterprise.
        External()

        // Adds a uni-directional relationship between this software system and the given element.
        Uses(ElementIdentifier, "<description>", "[technology]", Synchronous) /* or Asynchronous */)

        // Adds an interaction between this software system and a person.
        Delivers(PersonIdentifier, "<description>", "[technology]", Synchronous) /* or Asynchronous */)
    })

    // Container defines a container within a software system.
    var ContainerIdentifier = Container(SoftwareSystemIdentifier, "<name>",  "[description]",  "[technology]",  func() { // optional
        Tag("<name>",  "[name]") // as many tags as neede

        // URL where more information about this container can be found.
        URL("<url>")

        // Adds a uni-directional relationship between this container and the given element.
        Uses(ElementIdentifier, "<description>", "[technology]", Synchronous) /* or Asynchronous */

        // Adds an interaction between this container and a person.
        Delivers(PersonIdentifier, "<description>", "[technology]", Synchronous) /* or Asynchronous */
    })

    // Container may also refer to a Goa service in which case the name
    // and description are taken from the given service definition and
    // the technology is set to "Go and Goa v3".
    var ContainerIdentifier = Container(SoftwareSystemIdentifier, GoaServiceIdentifier, func() {
        // ... see above
    })

    // Component defines a component within a container.
    var ComponentIdentifier = Component(ContainerIdentifier, "<name>",  "[description]",  "[technology]",  func() { // optional
        Tag("<name>",  "[name]") // as many tags as neede

        // Adds a uni-directional relationship between this component and the given element.
        Uses(ElementIdentifier, "<description>", "[technology]", Synchronous /* or Asynchronous */)

        // Adds an interaction between this component and a person.
        Delivers(PersonIdentifier, "<description>", "[technology]", Synchronous /* or Asynchronous */)
    })

    // DeploymentEnvironment provides a way to define a deployment
    // environment (e.g. development, staging, production, etc).
    DeploymentEnvironment("<name>", func() {

        // DeploymentNode defines a deployment node. Deployment nodes can be
        // nested, so a deployment node can contain other deployment nodes.
        // A deployment node can also contain InfrastructureNode and
        // ContainerInstance elements.
        var DeploymentNodeIdentifier = DeploymentNode("<name>",  "[description]",  "[technology]",  func() { // optional
            Tag("<name>",  "[name]") // as many tags as needed

            // Instances sets the number of instances, defaults to 1.
            Instances(2)

            // URL where more information about this deployment node can be
            // found.
            URL("<url>") 

            // Adds a uni-directional relationship between this and another deployment node.
            Uses(DeploymentNodeIdentifier, "<description>", "[technology]", Synchronous /* or Asynchronous */)
        })

        // InfrastructureNode defines an infrastructure node, typically
        // something like a load balancer, firewall, DNS service, etc.
        var InfrastructureNodeIdentifier = InfrastructureNode(DeploymentNodeIdentifier, "<name>", "[description]", "[technology]", func() { // optional
            Tag("<name>",  "[name]") // as many tags as needed

            // URL where more information about this infrastructure node can be
            // found.
            URL("<url>") 

            // Adds a uni-directional relationship between this and
            // another deployment element (deployment node,
            // infrastructure node, or container instance).
            Uses(DeploymentElementIdentifier, "<description>", "[technology]", Synchronous /* or Asynchronous */)
        })

        // ContainerInstance defines an instance of the specified
        // container that is deployed on the parent deployment node.
        var ContainerInstanceIdentifier = ContainerInstance(identifier, func() { // optional
            Tag("<name>",  "[name]") // as many tags as needed

            // URL where more information about this container node can be
            // found.
            URL("<url>") 

            // Adds a uni-directional relationship between this and
            // another deployment element (deployment node,
            // infrastructure node, or container instance).
            Uses(DeploymentElementIdentifier, "<description>", "[technology]", Synchronous /* or Asynchronous */)

            // HealthCheck defines a HTTP-based health check for this
            // container instance.
            HealthCheck("<name>", func() {

                // URL is the health check URL/endpoint.
                URL("<url>")
                
                // Interval is the polling interval, in seconds.
                Interval(42)

                // Timeout after which a health check is deemed as failed,
                // in milliseconds.
                Timeout(42)

                // Header defines a header that should be sent with the
                // request.
                Header("<name>", "<value>")
            })
        })
    })

    // Views is optional and defines one or more views.
    Views(func() {
        
        // SystemLandscape defines a System Landscape view.
        SystemLandscape("[key]", "[description]", func() {

            // Title of this view.
            Title("<title>")

            // Include all people and software systems.
            IncludeAll()

            // Include given elements and relationships in view.
            Include(Identifier, Identifier) // as many identifiers as needed

            // Exclude given elements or relationships.
            Exclude(Identifier, Identifier)) // as many identifiers as needed

            // AutoLayout enables automatic layout mode for the diagram. The
            // first argument indicates the rank direction, it must be one of
            // RankTopBottom, RankBottomTop, RankLeftRight or RankRightLeft.
            AutoLayout(RankTopBottom, func() {

                // Separation between ranks in pixels
                RankSeparation(200)

                // Separation between nodes in the same rank in pixels
                NodeSeparation(200) 

                // Separation between edges in pixels
                EdgeSeparation(10) 

                // Create vertices during automatic layout.
                Vertices()
            }) 

            // AnimationStep defines an animation step consisting of the specified elements.
            AnimationStep(Identifier, Identifier)
            
            // PaperSize defines the paper size that should be used to render
            // the view. The possible values for the argument follow the
            // patterns A[0-6]_[Portrait|Landscape], Letter_[Portrait|Landscape]
            // or Legal_[Portrait_Landscape]. Alternatively the argument may be
            // one of Slide_4_3, Slide_16_9 or Slide_16_10.
            PaperSize(Slide_4_3)

            // Make enterprise boundary visible to differentiate internal
            // elements from external elements on the resulting diagram.
            EnterpriseBoundaryVisible()
        })

        SystemContext(SoftwareSystemIdentifier, "[key]", "[description]", func() {
            // ... same usage as SystemLandscape.
        })
        
        Container(SoftwareSystemIdentifier, "[key]", "[description]", func() {
            // ... same usage as SystemLandscape without EnterpriseBoundaryVisible.

            // Make software system boundaries visible for "external" containers
            // (those outside the software system in scope).
            SystemBoundariesVisible()
        })

        Component(ContainerIdentifier, "[key]", "[description]", func() {
            // ... same usage as SystemLandscape without EnterpriseBoundaryVisible.

            // Make container boundaries visible for "external" components
            // (those outside the container in scope).
            ContainerBoundariesVisible()
        })

        // Filtered defines a Filtered view on top of the specified view. The
        // base key specifies the key of the System Landscape, System Context,
        // Container, or Component view on which this filtered view should be
        // based.
        Filtered("<base key>", func() {
            // Set of tags to include or exclude (if Exclude() is used)
            // elements/relationships when rendering this filtered view.
            Tag("<tag>", "[tag]") // as many as needed

            // Exclude elements and relationships with the given tags.
            Exclude()
        }) 

        // Dynamic defines a Dynamic view for the specified scope. The first
        // argument defines the scope of the view, and therefore what can be
        // added to the view, as follows: 
        //  * Global scope: People and software systems.
        //  * Software system scope: People, other software systems, and
        //    containers belonging to the software system.
        //  * Container scope: People, other software systems, other
        //    containers, and components belonging to the container.
        Dynamic(Global, "[key]", "[description]", func() {

            // Title of this view.
            Title("<title>")

            // AutoLayout enables automatic layout mode for the diagram. The
            // first argument indicates the rank direction, it must be one of
            // RankTopBottom, RankBottomTop, RankLeftRight or RankRightLeft.
            AutoLayout(RankTopBottom, func() {

                // Separation between ranks in pixels
                RankSeparation(200)

                // Separation between nodes in the same rank in pixels
                NodeSeparation(200) 

                // Separation between edges in pixels
                EdgeSeparation(10) 

                // Create vertices during automatic layout.
                Vertices()
            }) 

            // PaperSize defines the paper size that should be used to render
            // the view. The possible values for the argument follow the
            // patterns A[0-6]_[Portrait|Landscape], Letter_[Portrait|Landscape]
            // or Legal_[Portrait_Landscape]. Alternatively the argument may be
            // one of Slide_4_3, Slide_16_9 or Slide_16_10.
            PaperSize(Slide_4_3)

            // Sequence of relationships that make up dynamic diagram.
            Relationship(Identifier, Identifier)
            Relationship(Identifier, Identifier)
            // ...
        })
          
        // Dynamic view on software system or container uses the corresponding
        // identifier as first argument.
        Dynamic(SoftwareSystemOrContainerIdentifier, "[key]", "[description]", func() {
            // see usage above
        })

        // Deployment defines a Deployment view for the specified scope and
        // deployment environment. The first argument defines the scope of the
        // view, and the second property defines the deployment environment. The
        // combination of these two arguments determines what can be added to
        // the view, as follows: 
        //  * Global scope: All deployment nodes, infrastructure nodes, and
        //    container instances within the deployment environment.
        //  * Software system scope: All deployment nodes and infrastructure
        //    nodes within the deployment environment. Container instances within
        //    the deployment environment that belong to the software system.
        Deployment(Global, "<environment name>", "[key]", "[description]", func() {
            // ... same usage as SystemLandscape without EnterpriseBoundaryVisible.
        })

        // Deployment on a software system uses the software system as first
        // argument.
        Deployment(SoftwareSystemIdentifier, "<environment name>", "[key]", "[description]", func() {
            // see usage above
        })

        // Element describes the position of an instance of a model element
        // (Person, Software System, Container or Component) in a View. The
        // first argument represents the x value, the second argument the y
        // value.
        Element(Identifier, 42, 42)

        // Relationship describes an instance of a model relationship in a View.
        // The SourceIdentifier and TargetIdentifier are used to identify the relationship.
        Relationship(SourceIdentifier, TargetIdentifier, func() {

            // Description used in dynamic views.
            Description("<description>")

            // Order of relationship in dynamic views, e.g. 1.0, 1.1, 2.0
            Order("<order>")

            // Vertices lists the x and y coordinate of the vertices used to
            // render the relationship. The number of arguments must be even.
            Vertices(10, 20, 10, 40)

            // Routing algorithm used when rendering relationship, one of
            // Direct, Curved or Orthogonal.
            Routing(Orthogonal)

            // Position of annotation along line; 0 (start) to 100 (end).
            Position(50)
        })

        // Styles is a wrapper for one or more element/relationship styles,
        // which are used when rendering diagrams.
        Styles(func() {

            // Element defines an element style. All nested properties (shape,
            // icon, etc) are optional, see Structurizr - Notation for more
            // details.
            Element("<tag>", func() {
                Shape("<Box|RoundedBox|Circle|Ellipse|Hexagon|Cylinder|Pipe|Person|Robot|Folder|WebBrowser|MobileDevicePortrait|MobileDeviceLandscape|Component>")
                Icon("<file>")
                Width(42)
                Height(42)
                Background("#<rrggbb>")
                Color("#<rrggbb>")
                Stroke("#<rrggbb>")
                FontSize(42)
                Boder("<solid|dashed|dotted>")
                Opacity(42) // Between 0 and 100
                Metadata(true)
                Description(true)
            })

            // Relationship defines a relationship style. All nested properties
            // (thickness, color, etc) are optional, see Structurizr - Notation
            // for more details.
            Relationship("<tag>", func() {
                Thickness(42)
                Color("#<rrggbb>")
                Dashed(true)
                Routing("<Direct|Orthogonal|Curved>")
                FontSize(42)
                Width(42)
                Position(42) // Between 0 and 100
                Opacity(42)  // Between 0 and 100
            })
        })

        // Themes specifies one or more themes that should be used when
        // rendering diagrams. See Structurizr - Themes for more details.
        Themes("<ThemeURL>", "[ThemeURL]") // as many theme URLs as needed

        // Branding defines custom branding that should be used when rendering
        // diagrams and documentation. See Structurizr - Branding for more
        // details.
        Branding(func() {
            Logo("<file>")
            Font("<name>", "[url]")
        })
    })
})
```
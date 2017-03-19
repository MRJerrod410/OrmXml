Object relational mapping (ORM) from C# classes to XML and back.

Create simple C# classes (much like JSON) who's objects can be converted to XML to save records or application settings. 
Later, the object can be re-hydrate from the XML and used like normal.

The motivation for this project was having to create custom XML files for different types of application settings. 
With EntityXml, you can create and work with settings using C# objects and never bother with writing any XML.

Example Use:

// Define the custom class

[Orm]   // Custom classes must be marked with the Orm attribute
class OrmTestClass
{
    public int ID { get; set; }   // Basic property types only (string, int, double, etc.)
    public Name Name { get; set; }
    public List<Item> Items { get; set; }   // collections must be of type 'List'
}

[Orm]
class Name
{
    public string First { get; set; }
    public string Last { get; set; }
}

[Orm]
class Item
{
    public string Name { get; set; }
    public double Price { get; set; }
}

[TestMethod]
public void TestMethod1()
{            
    // create the custom object
    OrmTestClass testObject = new OrmTestClass
    {
        ID = 19,
        Name = new Name
        {
            First = "Bob",
            Last = "Jones"
        },
        Items = new List<Item>
        {
            new Item { Name = "chair", Price = 9.99 },
            new Item { Name = "hat", Price = 0.97 }
        }
    };

    string path = "The full path to save the XML file";

    EntityXml<OrmTestClass> EtoX = new EntityXml<OrmTestClass>(path);

    EtoX.Save(testObject);  // Save the object to disk

    OrmTestClass hydratedTestObject = EtoX.Load();  // Hydrate the object from the XML

}

using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Xml.Linq;

namespace Orm2
{
    public class EntityXml<TEntity> where TEntity : class
    {
        private string m_Path;
        public EntityXml(string path)
        {
            m_Path = path;
        }

        public void Save(TEntity entity)
        {
            XElement XEle = Transform(entity);
            XDocument XDoc = new XDocument(XEle);
            XDoc.Save(m_Path);
        }

        public TEntity Load()
        {
            XDocument XDoc = XDocument.Load(m_Path);
            XElement XEle = XDoc.Root;
            TEntity entity = Transform(XEle);
            return entity;
        }

        public XElement Transform(TEntity entity)
        {
            Type type = typeof(TEntity);    // Get the class type of TEntity
            XElement Body = TypeToXElement(type, entity);   // The core XML elements created from TEntity

            return Body;
        }

        private XElement TypeToXElement(Type type, object sourceObject, string Name = null)
        {
            // Change type.Name to the variable name
            string ElementName = Name ?? type.Name;

            XElement XEle = new XElement(ElementName); // The class name

            PropertyInfo[] properties = type.GetProperties();   // the class properties

            foreach (PropertyInfo property in properties)   // Loop through each property
            {
                Type PropertyType = property.PropertyType;  // Get the Type for the property
                                
                if (PropertyType.Name == "List`1")  // If the Type is List`1, then we need to drill into the contained Type
                {
                    IList list = (IList)property.GetValue(sourceObject);    // Create an IList (List) object, so we can loop through each item
                    if (list == null) continue; // If the List is null, it has no data (this property is not used)

                    XElement ListElement = new XElement(property.Name); // The name of the List property
                    XEle.Add(ListElement);

                    Type genericTypeArgument = PropertyType.GenericTypeArguments[0];    // Get the contained Type
                    
                    foreach (var obj in list)   // Loop through each if the objects in the List
                    {
                        XElement SubElements = TypeToXElement(genericTypeArgument, obj, property.Name);    // Recursively call this function (TypeToXElement) to get the elements for that type
                        ListElement.Add(SubElements);
                    }                    
                }
                else if (PropertyType.GetCustomAttribute(typeof(OrmAttribute)) != null) // Test if this is a user defined class by checking for the Orm attribute
                {
                    XElement SubElements = TypeToXElement(PropertyType, property.GetValue(sourceObject), property.Name);
                    XEle.Add(SubElements);
                }
                else // The type is not List`1 or a custom class, then it is assumed to be a basic data type
                {
                    object obj = property.GetValue(sourceObject);
                    if (obj == null) continue;
                    XAttribute XAttr = new XAttribute(property.Name, obj);  // Add an attribute for this property
                    XEle.Add(XAttr);
                }
            }

            return XEle;
        }

        public TEntity Transform(XElement XEle)
        {
            Type EntityType = typeof(TEntity);
            TEntity entity = (TEntity)Activator.CreateInstance(EntityType);

            XElementToObject(EntityType, entity, XEle);

            return entity;
        }

        private object XElementToObject(Type SourceType, object SourceObj, XElement XEle)
        {
            PropertyInfo[] properties = SourceType.GetProperties();

            foreach (PropertyInfo property in properties)
            {
                Type PropertyType = property.PropertyType;
                
                if (PropertyType.Name == "List`1")  // If the Type is List`1, then we need to drill into the contained Type
                {
                    IEnumerable<XElement> SubXEles = XEle.Elements().Where(e => e.Name == property.Name).FirstOrDefault()?.Elements();

                    if (SubXEles == null) continue;

                    // Create list object
                    Type GenericTypeArgument = PropertyType.GenericTypeArguments[0];
                    IList CreatedObject = (IList)MakeObject(typeof(List<>), GenericTypeArgument);

                    property.SetValue(SourceObj, CreatedObject);

                    foreach (XElement el in SubXEles)
                    {
                        object sObj = MakeObject(GenericTypeArgument);
                        object obj = XElementToObject(GenericTypeArgument, sObj, el);
                        CreatedObject.Add(obj);
                    }
                }
                else if (PropertyType.GetCustomAttribute(typeof(OrmAttribute)) != null)
                {
                    XElement SubXEles = XEle.Elements().Where(e => e.Name == property.Name).FirstOrDefault();
                    if (SubXEles == null) continue;

                    object sObj = MakeObject(PropertyType);
                    object obj = XElementToObject(PropertyType, sObj, SubXEles);
                    property.SetValue(SourceObj,obj);
                }
                else
                {
                    XAttribute Attr = XEle.Attribute(property.Name);
                    if (Attr == null) continue;
                    object CreatedObject = Attr.Value;
                    property.SetValue(SourceObj, Convert.ChangeType(CreatedObject, PropertyType));
                }                
            }

            return SourceObj;
        }

        private object MakeObject(Type GenericType, Type genericTypeArgument)
        {
            Type type = GenericType.MakeGenericType(genericTypeArgument);
            IList obj = (IList)Activator.CreateInstance(type);
            return obj;
        }

        private object MakeObject(Type type)
        {
            object obj = Activator.CreateInstance(type);
            return obj;
        }
    }
}

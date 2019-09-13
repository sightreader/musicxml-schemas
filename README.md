## What is this for?

If you are writing a C# program that consumes MusicXML files (a digital format for sheet music), it may be helpful to deserialize MusicXML files to a strongly typed class instead of manually parsing the XML document. 

These C# classes help you do that. They are auto-generated from the official MusicXML schema definition files using the [XmlSchemaClassGenerator](https://github.com/mganss/XmlSchemaClassGenerator) project.

## Excerpt Sample

```cs
 ...
 ...
 ...

 /// <summary>
    /// <para>The score-partwise element is the root element for a partwise MusicXML score. It includes a score-header group followed by a series of parts with measures inside. The document-attributes attribute group includes the version attribute.</para>
    /// </summary>
    [System.CodeDom.Compiler.GeneratedCodeAttribute("XmlSchemaClassGenerator", "2.0.0.0")]
    [System.SerializableAttribute()]
    [System.Xml.Serialization.XmlTypeAttribute("score-partwise", Namespace="", AnonymousType=true)]
    [System.ComponentModel.DesignerCategoryAttribute("code")]
    [System.Xml.Serialization.XmlRootAttribute("score-partwise", Namespace="")]
    public partial class ScorePartwise
    {
        
        [System.Xml.Serialization.XmlElementAttribute("work", Namespace="")]
        public Work Work { get; set; }
        
        /// <summary>
        /// <para>The movement-number element specifies the number of a movement.</para>
        /// </summary>
        [System.Xml.Serialization.XmlElementAttribute("movement-number", Namespace="")]
        public string MovementNumber { get; set; }
        
        /// <summary>
        /// <para>The movement-title element specifies the title of a movement, not including its number.</para>
        /// </summary>
        [System.Xml.Serialization.XmlElementAttribute("movement-title", Namespace="")]
        public string MovementTitle { get; set; }
 ...
 ...
 ...
 28,500 more lines...
```

## Usage Example

I briefly tested each C# class against one MusicXML file sourced from MuseScore.org (ClassicMan's transcription of Chopin's Scherzo No. 2 Op. 31):

```cs
using System;
using System.Xml.Serialization;
using System.IO;

namespace ExampleProgram
{
    static class Program
    {
        static void Main()
        {
            XmlSerializer ser = new XmlSerializer(typeof(MusicXmlSchema.ScorePartwise));

            // Uncompress the MusicXML file if it ends with a .mxl before passing it in
            string filename = "scherzo-no-2-opus-31.xml";

            // Your fully typed score
            var score = ser.Deserialize(new FileStream(filename, FileMode.Open)) as MusicXmlSchema.ScorePartwise;

            Console.WriteLine(score.Credit[0].CreditWords.Value);
            // Output: Scherzo No. 2 in B♭ Mino
            Console.WriteLine(score.Part[0].Measure[0].Note[0].Pitch.Step);
            // Output: B

            Console.ReadKey();
        }
    }
}

```

## Folder Structure

Each folder contains:

- The original MusicXML schema definition files as downloaded from [musicxml.com](
https://www.musicxml.com/for-developers/musicxml-xsd/) for that version
- The converted XSD -> C# class for XML deserialization

## Generation

1. [Run XmlSchemaClassGenerator](https://github.com/sightreader/XmlSchemaClassGenerator/commit/957617e0a6602f774f9c68e3796c50f2eb4da62e#diff-962ecf59898691a00089b77166a0114aR26) on each MusicXML version of the `musicxml.xsd` files. 
2. Manually remove some extra generated classes `X_1`, `X_2` (there were 19 instances of these extra classes).

## Credits

Thanks to [XmlSchemaClassGenerator](https://github.com/mganss/XmlSchemaClassGenerator) for a great conversion library.

I tried other tools like .NET's built-in `xsd.exe` and the commercial `xsd2code`, but both `xsd.exe` and `xsd2code` do not pascal case names (they capitalizes the first letter but doesn't pascal case according to the dashes in element names).

The original `XmlSchemaClassGenerator` uses `_` underscores between words instead of plain pascal casing so [this fork](https://github.com/sightreader/XmlSchemaClassGenerator) has a quick patch to remove the `_` to allow plain pascal casing.

The generated C# classes also contained some repeat _1 _2 _3 suffix classes that caused errors when I tried the usage example so I manually removed those.
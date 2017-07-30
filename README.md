# AttributesSection
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using System.Data.Sql;
using System.Data;
using System.Data.SqlClient;
using System.Text;
using System.Web.UI.HtmlControls;
using System.Web.UI.WebControls.WebParts;


public partial class addattributes : System.Web.UI.Page
{
    public int Foreign_Key;

    private string connectionString = @"Data Source=.;Initial Catalog=UML;Integrated Security=True";

    private string finalString = "";

    private string relationshipFinalString = "";

    private List<int> ids = new List<int>();

    private List<string> entities = new List<string>();

    private List<string> attributesSections = new List<string>();

    private List<string> BehaviorsSections = new List<string>();

    private List<string> packages = new List<string>();

    private List<string> ERDpackages = new List<string>();

    private List<SqlDataReader> readers = new List<SqlDataReader>();

    private List<SqlCommand> commands = new List<SqlCommand>();

    private List<string> relationship = new List<string>();

    private string ClassDiagram;

    private int entityID;

    private bool KeyCheck = true;

    private int perojectID;

    private int FKentityID;

    private string FKattributeName = "";

    protected void Page_Load(object sender, EventArgs e)
    {
        //ddl_attributesReference.DataBind();
        entityID = Convert.ToInt16(Session["entityID"]);
        Label4.Text = entityID.ToString();
        Label4.Visible = false;
        GridView1.DataBind();

        perojectID = Convert.ToInt16(Session["ProjectID"]);
        Label3.Text = perojectID.ToString();
        Label3.Visible = false;

    }
    protected void btn_addattributes_Click(object sender, EventArgs e)
    {
        if (tb_attributesname.Text.Length >= 1)
        {
            string key = "";
            if (RadioButton_Primary.Checked == true)
            {
                key = "PRIMARY KEY";
            }
            else if (RadioButton_Foreign.Checked == true)
            {
                key = "FOREIGN KEY";
            }

            // select the attributes names of a specific entity to check the repeatation of data
            SqlConnection ObjConnection2 = new SqlConnection("Data Source=.;Initial Catalog=UML;Integrated Security=True");
            ObjConnection2.Open();
            string StringCmnd2 = @"SELECT Attribute.Attribute_Name FROM Attribute WHERE Attribute.Attribute_Name = '" + tb_attributesname.Text + "' AND Attribute.Entity_ID ='" + entityID + "'";
            SqlCommand SqlCmnd2 = new SqlCommand(StringCmnd2, ObjConnection2);
            SqlDataReader myDataReader2 = SqlCmnd2.ExecuteReader();
            if (myDataReader2.HasRows)
            {
                myDataReader2.Read();
                string myScriptValue = " {sweetAlert('Oops...', 'The Attribute You Entered is Already Exist in This Entity...!', 'error');}";
                ScriptManager.RegisterClientScriptBlock(this, this.GetType(), "myScriptName", myScriptValue, true);
                ObjConnection2.Close();
            }
            // insert the foreign key and the data on the behalf of different key conditions
            else
            {
                SqlConnection ObjConnection1 = new SqlConnection("Data Source=.;Initial Catalog=UML;Integrated Security=True");
                ObjConnection1.Open();
                string StringCmnd1 = "";
                if (key == "PRIMARY KEY")
                {
                    // checks that PRIMARY KEY is already exist or not
                    checkPK();
                    if (KeyCheck == true)
                    {
                        StringCmnd1 = @"INSERT INTO [dbo].[Attribute]
                           ([Attribute_Name]
                           ,[Attribute_Type]
                           ,[Entity_ID]
                           ,[Attribute_Key])
                        VALUES
                        ('" + tb_attributesname.Text + "','" + dl_attributetype.Text + "','" + entityID + "','" + key + "')";
                    }
                }

                else if (key == "FOREIGN KEY")
                {
                    checkFK();

                    if (KeyCheck == true)
                    {
                        getFKentityID();

                        getFKattributeName();

                        StringCmnd1 = @"INSERT INTO [dbo].[Attribute]
                           ([Attribute_Name]
                           ,[Attribute_Type]
                           ,[Entity_ID]
                           ,[Attribute_Key]
                           ,[Attribute_FK_Reference]
                           ,[FK_Attribute_Name])
                     VALUES
                     ('" + tb_attributesname.Text + "','" + dl_attributetype.Text + "','" + entityID + "','" + key + "','" + ddl_attributesReference.Text + "','" + FKattributeName + "')";
                    }
               }

                else
                {
                    StringCmnd1 = @"INSERT INTO [dbo].[Attribute]
           ([Attribute_Name]
           ,[Attribute_Type]
           ,[Entity_ID])
     VALUES
           ('" + tb_attributesname.Text + "','" + dl_attributetype.Text + "','" + entityID + "')";
                }

                if (KeyCheck == true)
                {
                    SqlCommand ObjCmnd1 = new SqlCommand(StringCmnd1, ObjConnection1);
                    ObjCmnd1.ExecuteNonQuery();
                    ObjConnection1.Close();
                    tb_attributesname.Text = "";
                    string myScriptValue = " { swal('Done', 'Attribute is Added', 'success');}";
                    ScriptManager.RegisterClientScriptBlock(this, this.GetType(), "myScriptName", myScriptValue, true);
                    //  Page_Load(sender, e);
                    // DropDownList1.DataBind();
                }
            }
        }
        else
        {
            ClientScript.RegisterStartupScript(this.GetType(), "alert", "alert('Attribute Name Should Not Be Empty')", true);
        }
        GridView1.DataBind();
    }
    protected void RadioButton_CheckedChanged(object sender, System.EventArgs e)
    {
        if (RadioButton_Foreign.Checked == true)
        {
            lbl_attributeReference.Visible = true;
            ddl_attributesReference.Visible = true;
        }
        else if (RadioButton_Primary.Checked == true || RadioButton_Notnull.Checked == true)
        {
            lbl_attributeReference.Visible = false;
            ddl_attributesReference.Visible = false;
        }
    }
    protected void GridView1_SelectedIndexChanged(object sender, EventArgs e)
    {

    }

    // checks that primary key is alredy exist or not
    protected void checkPK()
    {
        string key = "PRIMARY KEY";
        SqlConnection PKObjConnection = new SqlConnection("Data Source=.;Initial Catalog=UML;Integrated Security=True");
        PKObjConnection.Open();
        string PKStringCmnd = @"SELECT Attribute.Attribute_Key FROM Attribute where  Attribute.Attribute_Key = '" + key + "' AND Attribute.Entity_ID = '" + entityID + "'";
        SqlCommand PKSqlCmnd = new SqlCommand(PKStringCmnd, PKObjConnection);
        SqlDataReader PKmyDataReader = PKSqlCmnd.ExecuteReader();
        if (PKmyDataReader.HasRows)
        {
            PKmyDataReader.Read();
            PKObjConnection.Close();
            KeyCheck = false;
            key = "";
            string myScriptValue = " {sweetAlert('Oops...', 'Primary Key is Already Exist in This Entity...!', 'error');}";
            ScriptManager.RegisterClientScriptBlock(this, this.GetType(), "myScriptName", myScriptValue, true);
        }
    }

    // checks that foreign key is alredy exist or not
    protected void checkFK()
    {
        string key = "FOREIGN KEY";
        SqlConnection PKObjConnection = new SqlConnection("Data Source=.;Initial Catalog=UML;Integrated Security=True");
        PKObjConnection.Open();
        string PKStringCmnd = @"SELECT Attribute.Attribute_Key FROM Attribute where  Attribute.Attribute_Key = '" + key + "' AND Attribute.Entity_ID = '" + entityID + "'";
        SqlCommand PKSqlCmnd = new SqlCommand(PKStringCmnd, PKObjConnection);
        SqlDataReader PKmyDataReader = PKSqlCmnd.ExecuteReader();
        if (PKmyDataReader.HasRows)
        {
            PKmyDataReader.Read();
            PKObjConnection.Close();
            KeyCheck = false;
            key = "";
            string myScriptValue = " {sweetAlert('Oops...', 'Foreign Key is Already Exist in This Entity...!', 'error');}";
            ScriptManager.RegisterClientScriptBlock(this, this.GetType(), "myScriptName", myScriptValue, true);
        }
    }


    // get the foreign key entity id
    protected void getFKentityID()
    {
        SqlConnection PKObjConnection = new SqlConnection("Data Source=.;Initial Catalog=UML;Integrated Security=True");
        PKObjConnection.Open();
        string PKStringCmnd = @"SELECT Entity.Entity_ID FROM Entity WHERE Entity.Entity_Name = '" + ddl_attributesReference.Text + "' AND Entity.Project_ID = '" + perojectID + "'";
        SqlCommand PKSqlCmnd = new SqlCommand(PKStringCmnd, PKObjConnection);
        SqlDataReader PKmyDataReader = PKSqlCmnd.ExecuteReader();
        if (PKmyDataReader.HasRows)
        {
            PKmyDataReader.Read();
            FKentityID = Convert.ToInt16(PKmyDataReader[0]);
            PKObjConnection.Close();
            
        }
    }

    // get the foreign key entity attribute name that is primary key in that entity
    protected void getFKattributeName()
    {
        string key = "PRIMARY KEY";
        SqlConnection PKObjConnection = new SqlConnection("Data Source=.;Initial Catalog=UML;Integrated Security=True");
        PKObjConnection.Open();
        string PKStringCmnd = @"SELECT Attribute.Attribute_Name from Attribute where Attribute.Attribute_Key = '" + key + "' AND Attribute.Entity_ID = '" + FKentityID + "'";
        SqlCommand PKSqlCmnd = new SqlCommand(PKStringCmnd, PKObjConnection);
        SqlDataReader PKmyDataReader = PKSqlCmnd.ExecuteReader();
        if (PKmyDataReader.HasRows)
        {
            PKmyDataReader.Read();
            FKattributeName = PKmyDataReader[0].ToString();
            PKObjConnection.Close();
            key = "";
        }
        else
        {
            KeyCheck = false;
            string myScriptValue = " {sweetAlert('Oops...', 'Invalid Reference. Please Select Valid Reference...!', 'error');}";
            ScriptManager.RegisterClientScriptBlock(this, this.GetType(), "myScriptName", myScriptValue, true);
        }
    }
    protected void btn_showaddattributes_Click(object sender, EventArgs e)
    {
        Response.Redirect("addattributes.aspx");
    }
    protected void btn_showaddbehaviors_Click(object sender, EventArgs e)
    {
        Response.Redirect("addbehaviors.aspx");
    }
}

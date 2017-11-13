HomeController:


using MyDatatableCRUD.Models;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace MyDatatableCRUD.Controllers
{
    public class HomeController : Controller
    {
        public ActionResult Index()
        {
            return View();
        }
        public ActionResult GetEmployees()
        {
            using (MyDatabaseEntities dc = new MyDatabaseEntities())
            {
                var employees = dc.Employees.OrderBy(a => a.FirstName).ToList();
                return Json(new { data = employees }, JsonRequestBehavior.AllowGet);
            }
        }

        [HttpGet]
        public ActionResult Save(int id)
        {
            using(MyDatabaseEntities dc = new MyDatabaseEntities())
            {
                var v = dc.Employees.Where(a => a.EmployeeID == id).FirstOrDefault();
                return View(v);
            }
        }

        [HttpPost]
        public ActionResult Save(Employee emp)
        {
            bool status = false;
            if (ModelState.IsValid)
            {
                using(MyDatabaseEntities dc = new MyDatabaseEntities())
                {
                    if(emp.EmployeeID > 0)
                    {
                        //Edit
                        var v = dc.Employees.Where(a => a.EmployeeID == emp.EmployeeID).FirstOrDefault();
                        if(v != null)
                        {
                            v.FirstName = emp.FirstName;
                            v.LastName = emp.LastName;
                            v.EmailID = emp.EmailID;
                            v.City = emp.City;
                            v.Country = emp.Country;
                        }
                    }
                    else
                    {
                        //Save
                        dc.Employees.Add(emp);
                    }
                    dc.SaveChanges();
                    status = true;
                }
            }
            return new JsonResult { Data = new { status = status } };
        }

        [HttpGet]
        public ActionResult Delete(int id)
        {
            using(MyDatabaseEntities dc = new MyDatabaseEntities())
            {
                var v = dc.Employees.Where(a => a.EmployeeID == id).FirstOrDefault();
                if( v != null)
                {
                    return View(v);
                }
                else
                {
                    return HttpNotFound();
                }
            }
        }

        [HttpPost]
        [ActionName("Delete")]
        public ActionResult DeleteEmployee(int id)
        {
            bool status = false;
            using(MyDatabaseEntities dc = new MyDatabaseEntities())
            {
                var v = dc.Employees.Where(a => a.EmployeeID == id).FirstOrDefault();
                if(v != null)
                {
                    dc.Employees.Remove(v);
                    dc.SaveChanges();
                    status = true;
                }
            }
            return new JsonResult { Data = new { status = status } };
        }
    }
}





Models/txtended/Employee.cs


using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace MyDatatableCRUD.Models
{
    [MetadataType(typeof(EmployeeMetadata))]
    public partial class Employee
    {
    }
    public class EmployeeMetadata
    {
        [Required (AllowEmptyStrings = false , ErrorMessage = "Please Provide First Name.")]
        public string FirstName { get; set; }
        [Required (AllowEmptyStrings = false , ErrorMessage = "Please Provide Last Name.")]
        public string LastName { get; set; }

    }
}




index.cshtml




@{
    Layout = null;
}

<!DOCTYPE html>

<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Index</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" />
    <link rel="stylesheet" href="https://cdn.datatables.net/1.10.13/css/jquery.dataTables.min.css" />
    <link href="~/Content/themes/base/jquery-ui.min.css" rel="stylesheet" />
    <style>
        span.field-validation-error{
            color:red;
        }
    </style>
</head>
<body>
    <div style="width:90%; margin:0 auto" class="tablecontainer">
        <a class="popup btn btn-primary" href="/home/save/0" style="margin-bottom:20px;margin-top:20px;">Add New Employee</a>
        <table id="myDatatable">
            <thead>
                <tr>
                    <th>First Name</th>
                    <th>Last Name</th>
                    <th>Email ID</th>
                    <th>City</th>
                    <th>Country</th>
                    <th>Edit</th>
                    <th>Delete</th>
                </tr>
            </thead>
        </table>
    </div>
    <script src="~/scripts/jquery-3.2.1.min.js"></script>
    <script src="~/scripts/jquery.validate.min.js"></script>
    <script src="~/scripts/jquery.validate.unobtrusive.min.js"></script>
    <script src="http://cdn.datatables.net/1.10.13/js/jquery.dataTables.min.js"></script>
    <script src="~/scripts/jquery-ui-1.12.1.min.js"></script>
    <script>
        $(document).ready(function () {
            debugger
            var oTable = $('#myDatatable').DataTable({
                "ajax": {
                    "url": '/home/GetEmployees',
                    "type": "get",
                    "datatype": "json"
                },
                "columns": [
                    { "data": "FirstName", "autowidth": true },
                    { "data": "LastName", "autowidth": true },
                    { "data": "EmailID", "autowidth": true },
                    { "data": "City", "autowidth": true },
                    { "data": "Country", "autowidth": true },
                    {
                        "data": "EmployeeID", "width": "50px", "render": function (data) {
                            return '<a class="popup" href="/home/save/' + data + '">Edit</a>';
                        }
                    },
                    {
                        "data": "EmployeeID", "width": "50px", "render": function (data) {
                            return '<a class="popup" href="/home/delete/' + data + '">Delete</a>';
                        }
                    }
                ]
            })
        })
        $('.tablecontainer').on('click', 'a.popup', function (e) {
            e.preventDefault();
            OpenPopup($(this).attr('href'));
        })

        function OpenPopup(pageUrl) {
            var $pageContent = $('<div/>');
            $pageContent.load(pageUrl, function () {
                $('#popupForm', $pageContent).removeData('validator');
                $('#popupForm', $pageContent).removeData('unobtrusiveValidation');
                $.validator.unobtrusive.parse('form');
            });

            $dialog = $('<div class="popupWindow" style="overflow:auto"></div>')
            .html($pageContent)
            .dialog({
                draggable: false,
                autoOpen: false,
                resizable: false,
                model: true,
                title: "Popup Dialog",
                height: 550,
                width: 600,
                close: function () {
                    $dialog.dialog('destroy').remove();
                }
            })
            $('.popupWindow').on('submit', '#popupForm', function (e) {
                var url = $('#popupForm')[0].action;
                $.ajax({
                    type: "post",
                    url: url,
                    data: $('#popupForm').serialize(),
                    success: function (data) {
                        if (data.status) {
                            $dialog.dialog('close');
                            oTable.ajax.reload();
                        }
                    }
                })
                e.preventDefault();
            })
            $dialog.dialog('open');
        }
    </script>
</body>
</html>



Views/Home/Delete.cshtml


@model MyDatatableCRUD.Models.Employee

<h2>Delete Employee</h2>
@using (Html.BeginForm("delete" , "home" , FormMethod.Post, new { id = "popupForm" }))
{
    @Html.HiddenFor(a=>a.EmployeeID)
    <div class="form-group">
        <label>First Name</label>
        <p>@Model.FirstName</p>
    </div>
    <div class="form-group">
        <label>Last Name</label>
        <p>@Model.LastName</p>
    </div>
    <div class="form-group">
        <label>Email ID</label>
        <p>@Model.EmailID</p>
    </div>
    <div class="form-group">
        <label>City</label>
        <p>@Model.City</p>
    </div>
    <div class="form-group">
        <label>Country</label>
        <p>@Model.Country</p>
    </div>
    <div>
        <input type="submit" value="Delete" />
    </div>

}





Views/Home/Save.cshtml


@model MyDatatableCRUD.Models.Employee

<h2>Save</h2>
@using (Html.BeginForm("save", "home", FormMethod.Post, new { id = "popupForm" }))
{
    if(Model != null && Model.EmployeeID > 0)
    {
        @Html.HiddenFor(a=>a.EmployeeID)

    }
    <div class="form-group">
        <label>First Name</label>
        @Html.TextBoxFor(a => a.FirstName, new { @class = "form-control"})
        @Html.ValidationMessageFor(a=>a.FirstName)
    </div>
    <div class="form-group">
        <label>Last Name</label>
        @Html.TextBoxFor(a => a.LastName, new { @class = "form-control"})
        @Html.ValidationMessageFor(a=>a.LastName)
    </div>
    <div class="form-group">
        <label>Email ID</label>
        @Html.TextBoxFor(a=>a.EmailID, new { @class = "form-control"})
        @Html.ValidationMessageFor(a=>a.EmailID)
    </div>
    <div class="form-group">
        <label>City</label>
        @Html.TextBoxFor(a=>a.City, new { @class = "form-control" })
        @Html.ValidationMessageFor(a=>a.City)
    </div>
    <div class="form-group">
        <label>Country</label>
        @Html.TextBoxFor(a=>a.Country, new { @class = "form-control"})
        @Html.ValidationMessageFor(a=>a.Country)
    </div>

    <div>
        <input type="submit" value="Save" />
    </div>
}

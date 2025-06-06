# ASP.NET-MVC-Entity-Framework-Assignment-Part-4.cs
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace CarInsurance.Models
{
    public enum CoverageType   
    {
        Liability = 0,
        Full      = 1
    }

    public class Insuree
    {
        public int Id { get; set; }

       
        [Required, Display(Name = "First Name")]
        public string FirstName { get; set; }

        [Required, Display(Name = "Last Name")]
        public string LastName { get; set; }

        [Required, EmailAddress]
        public string EmailAddress { get; set; }

        [Required, DataType(DataType.Date)]
        [Display(Name = "Date of Birth")]
        public DateTime DateOfBirth { get; set; }

       
        [Required, Range(1900, 3000)]
        [Display(Name = "Car Year")]
        public int CarYear { get; set; }

        [Required]
        [Display(Name = "Car Make")]
        public string CarMake { get; set; }

        [Required]
        [Display(Name = "Car Model")]
        public string CarModel { get; set; }

        
        [Required, Range(0, int.MaxValue)]
        [Display(Name = "Number of Speeding Tickets")]
        public int SpeedingTickets { get; set; }

        [Required]
        [Display(Name = "Ever had a DUI?")]
        public bool HasDui { get; set; }

        
        [Required]
        [Display(Name = "Coverage Type")]
        public CoverageType CoverageType { get; set; }

       
        [ScaffoldColumn(false)]          
        [DataType(DataType.Currency)]
        public decimal Quote { get; set; }
    }
}


using System;
using System.Data.Entity;
using System.Linq;
using System.Net;
using System.Web.Mvc;
using CarInsurance.Models;

namespace CarInsurance.Controllers
{
    public class InsureeController : Controller
    {
        private readonly ApplicationDbContext _db = new ApplicationDbContext();

        
        public ActionResult Create()
        {
            return View();
        }

        
        [HttpPost, ValidateAntiForgeryToken]
        public ActionResult Create(Insuree insuree)
        {
            if (!ModelState.IsValid)
                return View(insuree);

            
            decimal quote = 50m; 

            
            int age = DateTime.Today.Year - insuree.DateOfBirth.Year;
            if (insuree.DateOfBirth.Date > DateTime.Today.AddYears(-age)) age--;

            if (age <= 18)
                quote += 100;
            else if (age <= 25)
                quote += 50;
            else
                quote += 25;

            
            if (insuree.CarYear < 2000)  quote += 25;
            if (insuree.CarYear > 2015)  quote += 25;

            
            bool isPorsche = insuree.CarMake.Equals("Porsche", StringComparison.OrdinalIgnoreCase);
            if (isPorsche)
            {
                quote += 25;
                if (insuree.CarModel.Equals("911 Carrera", StringComparison.OrdinalIgnoreCase))
                    quote += 25;
            }

            
            quote += insuree.SpeedingTickets * 10;

           
            if (insuree.HasDui)
                quote *= 1.25m;

            
            if (insuree.CoverageType == CoverageType.Full)
                quote *= 1.50m;

            insuree.Quote = Math.Round(quote, 2);

            
            _db.Insurees.Add(insuree);
            _db.SaveChanges();

            return RedirectToAction("Details", new { id = insuree.Id });
        }

        
        [Authorize(Roles = "Admin")]       
        public ActionResult Admin()
        {
            var allQuotes = _db.Insurees
                               .OrderByDescending(i => i.Id)
                               .ToList();
            return View(allQuotes);
        }

        
    }
}

@model CarInsurance.Models.Insuree
@{
    ViewBag.Title = "New Quote";
}

<h2>@ViewBag.Title</h2>

@using (Html.BeginForm()) 
{
    @Html.AntiForgeryToken()

    <div class="form-horizontal">
        <h4>Insuree</h4>
        <hr />
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })

        <!-- ALL the usual editors except Quote -->
        @Html.EditorForModel()   <!-- EditorForModel respects [ScaffoldColumn(false)] -->

        <div class="form-group">
            <div class="col-md-offset-2 col-md-10">
                <input type="submit" value="Get Quote" class="btn btn-primary" />
            </div>
        </div>
    </div>
}
@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
}

@model IEnumerable<CarInsurance.Models.Insuree>
@{
    ViewBag.Title = "All Quotes";
}

<h2>@ViewBag.Title</h2>

<table class="table table-striped table-bordered">
    <thead>
        <tr>
            <th>#</th>
            <th>First&nbsp;Name</th>
            <th>Last&nbsp;Name</th>
            <th>Email</th>
            <th>Quote&nbsp;(Monthly)</th>
            <th>Issued&nbsp;On (UTC)</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var i in Model)
        {
            <tr>
                <td>@i.Id</td>
                <td>@i.FirstName</td>
                <td>@i.LastName</td>
                <td>@i.EmailAddress</td>
                <td>@i.Quote.ToString("C")</td>
                <td>@i.Id</td> <!-- swap for CreatedAt if you add that field -->
            </tr>
        }
    </tbody>
</table>
routes.MapRoute(
    name: "AdminQuotes",
    url: "Admin",
    defaults: new { controller = "Insuree", action = "Admin" }
);



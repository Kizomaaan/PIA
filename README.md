# PIA

U frontendu:
npm install file-saver --save
import { saveAs } from 'file-saver';

sacuvajSablon(){
    const jsonRadionica = JSON.stringify(this.promenjenaRadionica);
    const blob = new Blob([jsonRadionica], { type: 'application/json' });
    saveAs(blob, 'radionica.json');
  }

pomocnaRadionica: Radionica;
  onFileSelected(event: any): void {
    const file: File = event.target.files[0];
    const reader: FileReader = new FileReader();
    reader.readAsText(file);
    reader.onload = () => {
      const jsonData = JSON.parse(reader.result as string);
      this.pomocnaRadionica = jsonData;
      this.naziv = this.pomocnaRadionica.naziv;
      this.datum =  this.pomocnaRadionica.datum;
      this.mesto = this.pomocnaRadionica.mesto;
      this.idOrganizatora =  this.pomocnaRadionica.idOrganizatora;
      //this.nizUcesnika =  this.pomocnaRadionica.nizUcesnika;
      this.kratakOpis =  this.pomocnaRadionica.kratakOpis;
      this.dugacakOpis =  this.pomocnaRadionica.dugacakOpis;   
      this.glavnaSlika = this.pomocnaRadionica.glavnaSlika;
      this.galerijaSlika = this.pomocnaRadionica.galerijaSlika;
    };
  }
HTML code za DodavanjeRadionicaStranica kako bi imali import json fajla:
<tr>
	<td>
              	<div class="group">      
                	<input type="file" (change)="onFileSelected($event)">
                	<span class="highlight"></span>
                	<span class="bar"></span>
                	<label class="aktivnaLabela">Import JSON sablona</label>
              	</div>
	</td>
</tr>

Takodje treba dodati f-ju sacuvajSablon() na click buttona za to 
U backednu:
npm install nodemailer
import UserModel from '../models/user'
const nodemailer = require('nodemailer')

// gamil app-password: bwzdtitgejylsion
    delete = async (req, res) => {
        console.log("Usao u delete");
      
        // Find the radionica document to delete
        const idRad = req.body.id;
        const radionica = await RadionicaModel.findOne({ id: idRad });
      
        if (!radionica) {
          res.status(404).json({ message: 'Radionica ne postoji' });
          return;
        }
      
        // Send email to each user in the nizUcesnika array
        const transporter = nodemailer.createTransport({
          service: 'gmail',
          auth: {
            user: 'nrdzoni7@gmail.com',
            pass: 'bwzdtitgejylsion'
          }
        });
      
        const promises = radionica.nizUcesnika.map(async (userId) => {
          try {
            const user = await UserModel.findOne({ id: userId });
      
            if (user) {
              const mailOptions = {
                from: 'nrdzoni7@gmail.com',
                to: user.email,
                subject: 'Radionica otkazana',
                text: `Postovani ${user.firstname} ${user.lastname}, obavestavamo Vas da je radionica ${radionica.naziv} otkazana.`
              };
      
              await transporter.sendMail(mailOptions);
            }
          } catch (error) {
            console.error(`Error sending email to user ${userId}: ${error}`);
          }
        });
      
        // Wait for all promises to resolve before deleting the radionica document
        Promise.all(promises).then(async () => {
            await RadionicaModel.deleteOne({ id: idRad });
            res.json({ message: 'Radionica deleted' });
        })
            .catch((error) => {
            console.error(`Error sending emails: ${error}`);
            res.status(500).json({ message: 'Error sending emails' });
        });
    };

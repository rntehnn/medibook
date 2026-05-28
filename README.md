import { useState, useCallback, useEffect } from "react";

type Lang = "fr" | "en" | "ro";
type Role = "patient" | "doctor";
type Status = "confirmed" | "cancelled";

interface Doctor { id:number;firstName:string;lastName:string;specialty:number;city:string;address:string;lat:number;lng:number;experience:number;rating:number;reviews:number;avatar:string;color:string;available:boolean;teleConsult:boolean;price:number; }
interface Slot { id:string;doctorId:number;date:string;time:string; }
interface User { id:number;firstName:string;lastName:string;email:string;password:string;phone:string;role:Role;specialty:number;city:string;address:string; }
interface Appointment { id:number;doctorId:number;patientId:number;patientName:string;doctorName:string;specialty:string;address:string;date:string;time:string;reason:string;status:Status; }

const T = {
  fr:{ heroTitle:"Trouvez votre médecin,",heroTitle2:"prenez RDV en secondes.",heroSub:"Consultation en cabinet ou téléconsultation, disponible 24h/24.",searchLabel:"Spécialité, médecin ou symptôme",searchBtn:"Rechercher",allSpec:"Toutes spécialités",doctors:"Médecins disponibles",howTitle:"Comment ça marche",s1t:"Cherchez",s1d:"Par spécialité, nom ou symptôme",s2t:"Choisissez",s2d:"Disponibilités en temps réel",s3t:"Confirmez",s3d:"Confirmation instantanée",sp:"Patients satisfaits",sd:"Médecins partenaires",sa:"RDV ce mois",sr:"Note moyenne",login:"Connexion",register:"S'inscrire",logout:"Déconnexion",email:"E-mail",password:"Mot de passe",firstName:"Prénom",lastName:"Nom",phone:"Téléphone",accType:"Type de compte",patient:"Patient",doctor:"Médecin",spec:"Spécialité",city:"Ville",address:"Adresse du cabinet",addrPh:"Ex : 12 rue de la Paix, 75001 Paris",noAcc:"Pas encore de compte ?",hasAcc:"Déjà un compte ?",welcome:"Content de vous revoir",dash:"Mon espace",myAppts:"Mes RDV",findDoc:"Trouver un médecin",book:"Prendre RDV",cancel:"Annuler",confirm:"Confirmer",upcoming:"À venir",past:"Passés",noAppts:"Aucun rendez-vous",confirmed:"Confirmé",cancelled:"Annulé",slots:"Créneaux disponibles",noSlots:"Aucun créneau",reason:"Motif",reasonPh:"Décrivez brièvement…",booked:"Rendez-vous confirmé !",nextAppt:"Prochain RDV",totalPat:"Patients total",today:"Aujourd'hui",at:"à",exp:"ans d'exp.",reviews:"avis",tele:"Téléconsultation",avail:"Disponible aujourd'hui",bookWith:"Réservation avec",selSlot:"Choisissez un créneau",confStep:"Confirmer le RDV",cabinet:"Adresse",footerTag:"La santé accessible à tous.",about:"À propos",contact:"Contact",legal:"Mentions légales",back:"← Retour",langs:{fr:"FR",en:"EN",ro:"RO"},specs:["Généraliste","Cardiologue","Dermatologue","Pédiatre","Ophtalmologue","Gynécologue","Orthopédiste","Neurologue","Psychiatre","ORL"] },
  en:{ heroTitle:"Find your doctor,",heroTitle2:"book in seconds.",heroSub:"In-office or teleconsultation, available 24/7.",searchLabel:"Specialty, doctor or symptom",searchBtn:"Search",allSpec:"All specialties",doctors:"Available doctors",howTitle:"How it works",s1t:"Search",s1d:"By specialty, name or symptom",s2t:"Choose",s2d:"Real-time availability",s3t:"Confirm",s3d:"Instant confirmation",sp:"Happy patients",sd:"Partner doctors",sa:"Appts this month",sr:"Average rating",login:"Sign in",register:"Sign up",logout:"Log out",email:"Email",password:"Password",firstName:"First name",lastName:"Last name",phone:"Phone",accType:"Account type",patient:"Patient",doctor:"Doctor",spec:"Specialty",city:"City",address:"Practice address",addrPh:"E.g: 12 Baker St, London",noAcc:"No account yet?",hasAcc:"Already have an account?",welcome:"Welcome back",dash:"My space",myAppts:"My appointments",findDoc:"Find a doctor",book:"Book",cancel:"Cancel",confirm:"Confirm",upcoming:"Upcoming",past:"Past",noAppts:"No appointments",confirmed:"Confirmed",cancelled:"Cancelled",slots:"Available slots",noSlots:"No slots",reason:"Reason",reasonPh:"Briefly describe…",booked:"Appointment confirmed!",nextAppt:"Next appointment",totalPat:"Total patients",today:"Today",at:"at",exp:"yrs exp.",reviews:"reviews",tele:"Teleconsult",avail:"Available today",bookWith:"Booking with",selSlot:"Select a time slot",confStep:"Confirm appointment",cabinet:"Address",footerTag:"Healthcare for everyone.",about:"About",contact:"Contact",legal:"Legal",back:"← Back",langs:{fr:"FR",en:"EN",ro:"RO"},specs:["GP","Cardiologist","Dermatologist","Pediatrician","Ophthalmologist","Gynecologist","Orthopedist","Neurologist","Psychiatrist","ENT"] },
  ro:{ heroTitle:"Găsește medicul tău,",heroTitle2:"programează-te în secunde.",heroSub:"Consultație la cabinet sau teleconsultație, disponibil 24/7.",searchLabel:"Specialitate, medic sau simptom",searchBtn:"Caută",allSpec:"Toate specialitățile",doctors:"Medici disponibili",howTitle:"Cum funcționează",s1t:"Caută",s1d:"După specialitate sau nume",s2t:"Alege",s2d:"Disponibilitate în timp real",s3t:"Confirmă",s3d:"Confirmare instantanee",sp:"Pacienți mulțumiți",sd:"Medici parteneri",sa:"Programări luna aceasta",sr:"Nota medie",login:"Autentificare",register:"Înregistrare",logout:"Ieșire",email:"E-mail",password:"Parolă",firstName:"Prenume",lastName:"Nume",phone:"Telefon",accType:"Tip cont",patient:"Pacient",doctor:"Medic",spec:"Specialitate",city:"Oraș",address:"Adresa cabinetului",addrPh:"Ex: Str. Victoriei 12, București",noAcc:"Nu ai cont?",hasAcc:"Ai deja cont?",welcome:"Bine ai revenit",dash:"Spațiul meu",myAppts:"Programările mele",findDoc:"Găsește medic",book:"Programare",cancel:"Anulează",confirm:"Confirmă",upcoming:"Viitoare",past:"Trecute",noAppts:"Nicio programare",confirmed:"Confirmat",cancelled:"Anulat",slots:"Intervale disponibile",noSlots:"Niciun interval",reason:"Motiv",reasonPh:"Descrieți pe scurt…",booked:"Programare confirmată!",nextAppt:"Următoarea programare",totalPat:"Total pacienți",today:"Azi",at:"la",exp:"ani exp.",reviews:"recenzii",tele:"Teleconsultație",avail:"Disponibil azi",bookWith:"Programare cu",selSlot:"Selectați un interval",confStep:"Confirmați programarea",cabinet:"Adresă",footerTag:"Sănătate accesibilă pentru toți.",about:"Despre noi",contact:"Contact",legal:"Mențiuni legale",back:"← Înapoi",langs:{fr:"FR",en:"EN",ro:"RO"},specs:["Generalist","Cardiolog","Dermatolog","Pediatru","Oftalmolog","Ginecolog","Ortoped","Neurolog","Psihiatru","ORL"] }
};

const DOCTORS: Doctor[] = [
  {id:1,firstName:"Sophie",lastName:"Martin",specialty:0,city:"Paris",address:"12 rue de Rivoli, 75001 Paris",lat:48.8602,lng:2.3477,experience:12,rating:4.9,reviews:248,avatar:"SM",color:"#2D6A4F",available:true,teleConsult:true,price:25},
  {id:2,firstName:"Pierre",lastName:"Dubois",specialty:1,city:"Lyon",address:"8 place Bellecour, 69002 Lyon",lat:45.7578,lng:4.8320,experience:20,rating:4.8,reviews:312,avatar:"PD",color:"#1B4F72",available:true,teleConsult:false,price:50},
  {id:3,firstName:"Claire",lastName:"Bernard",specialty:2,city:"Paris",address:"45 av. Montaigne, 75008 Paris",lat:48.8656,lng:2.3050,experience:8,rating:4.7,reviews:189,avatar:"CB",color:"#6C3483",available:false,teleConsult:true,price:40},
  {id:4,firstName:"Marc",lastName:"Lefebvre",specialty:3,city:"Marseille",address:"3 rue Paradis, 13001 Marseille",lat:43.2965,lng:5.3698,experience:15,rating:4.9,reviews:421,avatar:"ML",color:"#A04000",available:true,teleConsult:true,price:30},
  {id:5,firstName:"Julie",lastName:"Moreau",specialty:4,city:"Bordeaux",address:"17 cours de l'Intendance, 33000 Bordeaux",lat:44.8378,lng:-0.5792,experience:10,rating:4.6,reviews:156,avatar:"JM",color:"#1A5276",available:true,teleConsult:false,price:45},
  {id:6,firstName:"Antoine",lastName:"Simon",specialty:5,city:"Nantes",address:"22 rue Crébillon, 44000 Nantes",lat:47.2184,lng:-1.5536,experience:18,rating:4.8,reviews:287,avatar:"AS",color:"#922B21",available:false,teleConsult:true,price:35},
];

const genSlots = (doctorId: number): Slot[] => {
  const slots: Slot[] = [];
  const base = new Date();
  for (let d=1;d<=7;d++) {
    const dt = new Date(base); dt.setDate(base.getDate()+d);
    if ([0,6].includes(dt.getDay())) continue;
    ["09:00","09:30","10:00","10:30","11:00","14:00","14:30","15:00","15:30","16:00"].forEach((t,i)=>{
      if (Math.random()>0.4) slots.push({id:`${doctorId}-${d}-${i}`,doctorId,date:dt.toISOString().split("T")[0],time:t});
    });
  }
  return slots;
};
const ALL_SLOTS: Slot[] = DOCTORS.flatMap(d=>genSlots(d.id));

const ls = {
  get:<T>(k:string,fb:T):T=>{ try{ return JSON.parse(localStorage.getItem(k)??"null")??fb; }catch{ return fb; } },
  set:(k:string,v:unknown)=>{ try{ localStorage.setItem(k,JSON.stringify(v)); }catch{} },
};

// Detect language from IP via free API
async function detectLang(): Promise<Lang> {
  try {
    const r = await fetch("https://ipapi.co/json/");
    const d = await r.json();
    const cc: string = d.country_code || "";
    if (cc === "FR" || cc === "BE" || cc === "CH" || cc === "LU" || cc === "MC" || cc === "CA") return "fr";
    if (cc === "RO" || cc === "MD") return "ro";
    return "en";
  } catch {
    return "en";
  }
}

const CSS = `
@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
:root{
  --ink:#0D1117;--ink2:#1C2333;--slate:#4A5568;--muted:#718096;
  --border:#E2E8F0;--bg:#F8FAFC;--w:#fff;
  --teal:#0D9488;--teal2:#14B8A6;--teal3:#CCFBF1;
  --red:#EF4444;--red2:#FEE2E2;--green2:#D1FAE5;--amber:#F59E0B;
  --r8:8px;--r12:12px;--r16:16px;--r24:24px;--r32:32px;
  --sd:0 4px 16px rgba(0,0,0,.08);--sdm:0 10px 40px rgba(0,0,0,.1);--sdl:0 24px 64px rgba(0,0,0,.12);
  --D:'Playfair Display',serif;--B:'Plus Jakarta Sans',sans-serif;
}
html{scroll-behavior:smooth}
body{font-family:var(--B);background:var(--bg);color:var(--ink);-webkit-font-smoothing:antialiased;min-height:100vh}
::-webkit-scrollbar{width:5px}::-webkit-scrollbar-thumb{background:var(--border);border-radius:9px}

/* NAV */
.nav{position:sticky;top:0;z-index:200;background:rgba(255,255,255,.92);backdrop-filter:blur(20px);border-bottom:1px solid var(--border);height:68px;display:flex;align-items:center;padding:0 clamp(1rem,4vw,3rem);gap:1.5rem}
.nlogo{font-family:var(--D);font-weight:900;font-size:1.5rem;color:var(--ink);cursor:pointer;border:none;background:none;display:flex;align-items:center;gap:.3rem;letter-spacing:-.5px;flex-shrink:0}
.ndot{width:8px;height:8px;border-radius:50%;background:var(--teal);margin-bottom:1px;display:inline-block}
.nmid{flex:1;display:flex;justify-content:center;gap:1.75rem}
.nl{font-size:.875rem;font-weight:500;color:var(--slate);cursor:pointer;background:none;border:none;font-family:var(--B);padding:4px 0;position:relative;transition:color .2s}
.nl::after{content:'';position:absolute;bottom:-2px;left:0;width:0;height:2px;background:var(--teal);border-radius:2px;transition:width .25s}
.nl:hover{color:var(--ink)}.nl:hover::after,.nl.act::after{width:100%}.nl.act{color:var(--ink)}
.nr{display:flex;align-items:center;gap:.75rem;margin-left:auto;flex-shrink:0}
.lp{display:flex;background:#EDF2F7;border-radius:99px;padding:3px;gap:2px}
.lb{padding:4px 10px;border-radius:99px;font-size:.74rem;font-weight:700;cursor:pointer;border:none;background:none;color:var(--muted);font-family:var(--B);transition:all .2s;letter-spacing:.3px}
.lb.on{background:var(--w);color:var(--ink);box-shadow:0 1px 3px rgba(0,0,0,.08)}

/* BUTTONS */
.btn{display:inline-flex;align-items:center;justify-content:center;gap:.5rem;padding:10px 22px;border-radius:var(--r12);font-size:.875rem;font-family:var(--B);font-weight:600;cursor:pointer;transition:all .2s;border:none;white-space:nowrap}
.bp{background:var(--ink);color:#fff}.bp:hover{background:var(--ink2);transform:translateY(-1px);box-shadow:0 4px 18px rgba(13,17,23,.22)}
.bt{background:var(--teal);color:#fff}.bt:hover{background:var(--teal2);transform:translateY(-1px);box-shadow:0 4px 18px rgba(13,148,136,.28)}
.bo{background:transparent;border:1.5px solid rgba(255,255,255,.25);color:#fff}.bo:hover{background:rgba(255,255,255,.12)}
.bg{background:none;border:none;color:var(--slate);padding:8px 14px}.bg:hover{color:var(--ink);background:#EDF2F7}
.bsm{padding:7px 16px;font-size:.82rem;border-radius:var(--r8)}
.blg{padding:14px 32px;font-size:1rem;border-radius:var(--r16)}
.bw{width:100%}
.bdan{background:var(--red2);color:var(--red)}.bdan:hover{background:var(--red);color:#fff}
.btn:disabled{opacity:.45;cursor:not-allowed;transform:none!important}

/* HERO */
.hero{min-height:calc(100vh - 68px);background:var(--ink);color:#fff;display:grid;grid-template-columns:1fr 1fr;align-items:center;padding:0 clamp(1.5rem,5vw,4rem);gap:4rem;position:relative;overflow:hidden}
.hbg{position:absolute;inset:0;background:radial-gradient(ellipse 80% 60% at 70% 50%,rgba(13,148,136,.18) 0%,transparent 70%),radial-gradient(ellipse 50% 80% at 10% 80%,rgba(201,168,76,.08) 0%,transparent 60%)}
.hgrid{position:absolute;inset:0;opacity:.04;background-image:linear-gradient(var(--border) 1px,transparent 1px),linear-gradient(90deg,var(--border) 1px,transparent 1px);background-size:40px 40px}
.hcnt{position:relative;z-index:1}
.hbadge{display:inline-flex;align-items:center;gap:.5rem;background:rgba(13,148,136,.15);border:1px solid rgba(13,148,136,.3);color:var(--teal2);padding:6px 14px;border-radius:99px;font-size:.8rem;font-weight:600;margin-bottom:2rem;letter-spacing:.3px}
.hbd{width:6px;height:6px;border-radius:50%;background:var(--teal2);animation:pu 2s infinite}
@keyframes pu{0%,100%{opacity:1;transform:scale(1)}50%{opacity:.5;transform:scale(1.3)}}
.hh1{font-family:var(--D);font-size:clamp(2.5rem,5vw,4rem);font-weight:900;line-height:1.1;letter-spacing:-1.5px;margin-bottom:1.5rem}
.hh1 em{font-style:normal;color:var(--teal2)}
.hsub{font-size:1rem;color:rgba(255,255,255,.6);line-height:1.7;margin-bottom:2.5rem;max-width:480px}
.hcta{display:flex;gap:1rem;flex-wrap:wrap}
.hstats{display:flex;gap:2rem;margin-top:3rem;padding-top:2rem;border-top:1px solid rgba(255,255,255,.1);flex-wrap:wrap}
.hsv{font-family:var(--D);font-size:1.8rem;font-weight:700;color:#fff}
.hsl{font-size:.78rem;color:rgba(255,255,255,.5);margin-top:2px}
.hvis{position:relative;z-index:1;display:flex;flex-direction:column;gap:1.25rem}
.hcard{background:rgba(255,255,255,.07);backdrop-filter:blur(20px);border:1px solid rgba(255,255,255,.12);border-radius:var(--r24);padding:1.5rem;color:#fff;animation:fc 6s ease-in-out infinite}
.hcard:nth-child(2){animation-delay:-2s;margin-left:2rem}
.hcard:nth-child(3){animation-delay:-4s;margin-left:1rem}
@keyframes fc{0%,100%{transform:translateY(0)}50%{transform:translateY(-8px)}}
.hcrow{display:flex;align-items:center;gap:1rem}
.havt{width:40px;height:40px;border-radius:var(--r8);display:flex;align-items:center;justify-content:center;font-weight:800;font-size:.8rem;color:#fff;flex-shrink:0}
.hcn{font-weight:700;font-size:.95rem}.hcs{font-size:.78rem;opacity:.6;margin-top:2px}
.hslots{display:flex;gap:.5rem;margin-top:1rem;flex-wrap:wrap}
.hsl2{background:rgba(13,148,136,.25);border:1px solid rgba(13,148,136,.4);color:var(--teal2);padding:5px 12px;border-radius:var(--r8);font-size:.78rem;font-weight:600}

/* SEARCH */
.sw{background:var(--w);padding:1.5rem clamp(1.5rem,4vw,3rem);border-bottom:1px solid var(--border);box-shadow:0 1px 3px rgba(0,0,0,.05)}
.sb{display:flex;align-items:center;background:var(--bg);border:1.5px solid var(--border);border-radius:var(--r16);overflow:hidden;transition:border-color .2s,box-shadow .2s}
.sb:focus-within{border-color:var(--teal);box-shadow:0 0 0 4px rgba(13,148,136,.1)}
.sf{display:flex;align-items:center;gap:.75rem;flex:1;padding:13px 20px;border-right:1px solid var(--border)}
.sf input{border:none;background:none;outline:none;font-size:.95rem;font-family:var(--B);color:var(--ink);width:100%}
.sf input::placeholder{color:var(--muted)}
.ssw{display:flex;align-items:center;gap:.5rem;padding:0 16px}
.ss{border:none;background:none;outline:none;font-size:.9rem;font-family:var(--B);color:var(--ink);cursor:pointer;padding:13px 4px}
.sbw{padding:8px}
.chips{display:flex;gap:.5rem;margin-top:.75rem;flex-wrap:wrap}
.chip{padding:6px 14px;border-radius:99px;font-size:.8rem;font-weight:500;cursor:pointer;border:1.5px solid var(--border);color:var(--slate);background:var(--w);transition:all .2s;font-family:var(--B)}
.chip:hover{border-color:var(--teal);color:var(--teal);background:var(--teal3)}

/* SECTIONS */
.sec{padding:72px clamp(1.5rem,4vw,3rem)}
.sech{display:flex;justify-content:space-between;align-items:flex-end;margin-bottom:2.5rem}
.sect{font-family:var(--D);font-size:clamp(1.5rem,3vw,2rem);font-weight:700;letter-spacing:-.5px}
.secs{color:var(--muted);font-size:.88rem;margin-top:.4rem}

/* HOW */
.how{background:var(--ink);color:#fff;padding:72px clamp(1.5rem,4vw,3rem)}
.howg{display:grid;grid-template-columns:repeat(3,1fr);gap:2rem;margin-top:3rem}
.howc{background:rgba(255,255,255,.05);border:1px solid rgba(255,255,255,.1);border-radius:var(--r24);padding:2rem;transition:background .3s}
.howc:hover{background:rgba(255,255,255,.08)}
.hown{font-family:var(--D);font-size:3rem;font-weight:900;color:rgba(255,255,255,.1);line-height:1;margin-bottom:1.5rem}
.howi{width:48px;height:48px;border-radius:var(--r12);background:rgba(13,148,136,.2);border:1px solid rgba(13,148,136,.3);display:flex;align-items:center;justify-content:center;font-size:1.3rem;margin-bottom:1.25rem}
.howt{font-family:var(--D);font-size:1.2rem;font-weight:700;margin-bottom:.5rem}
.howd{color:rgba(255,255,255,.5);font-size:.88rem;line-height:1.6}

/* DOCTOR CARD */
.dgrid{display:grid;grid-template-columns:repeat(auto-fill,minmax(290px,1fr));gap:1.5rem}
.dc{background:var(--w);border-radius:var(--r24);border:1px solid var(--border);overflow:hidden;transition:all .3s cubic-bezier(.34,1.56,.64,1);cursor:pointer}
.dc:hover{transform:translateY(-6px);box-shadow:var(--sdm);border-color:transparent}
.dct{padding:1.5rem 1.5rem 1rem;display:flex;gap:1rem;align-items:flex-start}
.da{width:54px;height:54px;border-radius:var(--r16);display:flex;align-items:center;justify-content:center;font-weight:800;font-size:1rem;flex-shrink:0;color:#fff;font-family:var(--D)}
.dn{font-family:var(--D);font-weight:700;font-size:1.05rem;line-height:1.2}
.dsp{color:var(--teal);font-size:.8rem;font-weight:600;margin-top:3px;text-transform:uppercase;letter-spacing:.4px}
.dci{color:var(--muted);font-size:.8rem;margin-top:2px}
.dtags{padding:0 1.5rem;display:flex;gap:.5rem;flex-wrap:wrap;margin-bottom:1rem}
.tag{padding:4px 10px;border-radius:99px;font-size:.74rem;font-weight:600}
.ta{background:var(--green2);color:#065F46}.tt{background:#EDE9FE;color:#5B21B6}.tn{background:#EDF2F7;color:var(--muted)}
.dmeta{padding:0 1.5rem 1.5rem;display:flex;justify-content:space-between;align-items:center}
.dstars{color:var(--amber);font-size:.85rem}
.dr2{font-weight:700;font-size:.88rem}.dc3{color:var(--muted);font-size:.76rem}
.dex{color:var(--muted);font-size:.8rem;margin-top:3px}
.dpr{font-size:.8rem;color:var(--muted)}.dpr strong{color:var(--ink);font-size:.95rem}
.dfoot{padding:1.1rem 1.5rem;border-top:1px solid var(--border);display:flex;justify-content:space-between;align-items:center;background:var(--bg)}
.dnx{font-size:.78rem;color:var(--muted)}.dnx strong{color:var(--teal)}

/* MODAL */
.ov{position:fixed;inset:0;background:rgba(13,17,23,.7);display:flex;align-items:flex-end;justify-content:center;z-index:500;backdrop-filter:blur(8px);animation:fi .2s ease}
@media(min-width:640px){.ov{align-items:center;padding:1rem}}
@keyframes fi{from{opacity:0}to{opacity:1}}
.modal{background:var(--w);width:100%;max-width:700px;max-height:92vh;overflow:hidden;display:flex;flex-direction:column;border-radius:var(--r32) var(--r32) 0 0;animation:su .3s cubic-bezier(.34,1.2,.64,1);box-shadow:var(--sdl)}
@media(min-width:640px){.modal{border-radius:var(--r32);max-height:88vh}}
@keyframes su{from{opacity:0;transform:translateY(40px)}to{opacity:1;transform:translateY(0)}}
.mhandle{width:40px;height:4px;background:var(--border);border-radius:2px;margin:12px auto 0}
.mh{padding:1.25rem 2rem;border-bottom:1px solid var(--border);display:flex;justify-content:space-between;align-items:center;flex-shrink:0}
.mt2{font-family:var(--D);font-weight:700;font-size:1.2rem}
.mx{width:36px;height:36px;border-radius:50%;border:none;background:#EDF2F7;display:flex;align-items:center;justify-content:center;cursor:pointer;font-size:1.2rem;color:var(--slate);transition:all .2s}
.mx:hover{background:var(--border)}
.mb2{flex:1;overflow-y:auto;padding:1.5rem 2rem}
.mf{padding:1.25rem 2rem;border-top:1px solid var(--border);display:flex;justify-content:flex-end;gap:.75rem;flex-shrink:0;background:var(--bg)}

/* MAP */
.mapw{border-radius:var(--r16);overflow:hidden;border:1px solid var(--border);height:200px}
.mapif{width:100%;height:100%;border:none}

/* SLOTS */
.slsec{margin-top:1.5rem}
.slt{font-size:.74rem;font-weight:700;text-transform:uppercase;letter-spacing:.8px;color:var(--muted);margin-bottom:1rem}
.sld{margin-bottom:1.25rem}
.sldl{font-size:.82rem;font-weight:600;color:var(--slate);margin-bottom:.6rem;display:flex;align-items:center;gap:.5rem}
.sldl::after{content:'';flex:1;height:1px;background:var(--border)}
.slrow{display:flex;flex-wrap:wrap;gap:.5rem}
.slot{padding:8px 16px;border-radius:var(--r8);font-size:.82rem;font-weight:600;cursor:pointer;border:1.5px solid var(--border);color:var(--ink);background:#fff;transition:all .15s;font-family:var(--B)}
.slot:hover{border-color:var(--teal);color:var(--teal);background:var(--teal3)}
.slot.on{background:var(--ink);color:#fff;border-color:var(--ink)}

/* CONFIRM */
.cfc{background:linear-gradient(135deg,var(--teal) 0%,#0F766E 100%);color:#fff;border-radius:20px;padding:1.5rem;margin-bottom:1.5rem;display:flex;align-items:center;gap:1rem}
.cfn{font-family:var(--D);font-size:1.1rem;font-weight:700}
.cfs{opacity:.8;font-size:.88rem;margin-top:3px}
.icrd{background:var(--bg);border-radius:var(--r16);padding:1.25rem;border:1px solid var(--border)}
.ict{font-size:.74rem;font-weight:700;text-transform:uppercase;letter-spacing:.8px;color:var(--muted);margin-bottom:1rem}
.ir{display:flex;align-items:flex-start;gap:.75rem;padding:.6rem 0;border-bottom:1px solid var(--border);font-size:.88rem}
.ir:last-child{border-bottom:none;padding-bottom:0}

/* AUTH */
.aw{min-height:calc(100vh - 68px);display:flex;align-items:center;justify-content:center;padding:2rem;background:linear-gradient(135deg,var(--bg) 0%,#E0F2FE 100%)}
.ac{background:var(--w);border-radius:var(--r32);padding:2.5rem;width:100%;max-width:450px;box-shadow:var(--sdl);border:1px solid var(--border)}
.alogo{text-align:center;margin-bottom:1.5rem;font-family:var(--D);font-weight:900;font-size:1.5rem;cursor:pointer;border:none;background:none;width:100%}
.at{font-family:var(--D);font-size:1.75rem;font-weight:800;margin-bottom:.4rem}
.as2{color:var(--muted);font-size:.9rem;margin-bottom:2rem}
.atg{text-align:center;margin-top:1.25rem;font-size:.88rem;color:var(--muted)}
.atg a{color:var(--teal);cursor:pointer;font-weight:700}

/* FORMS */
.fg{margin-bottom:1.1rem}
.lbl{display:block;font-size:.82rem;font-weight:700;color:var(--slate);margin-bottom:6px}
.inp,.sel,.tex{width:100%;padding:11px 14px;border:1.5px solid var(--border);border-radius:var(--r12);font-size:.92rem;font-family:var(--B);outline:none;transition:all .2s;background:#fff;color:var(--ink)}
.inp:focus,.sel:focus,.tex:focus{border-color:var(--teal);box-shadow:0 0 0 4px rgba(13,148,136,.1)}
.tex{resize:vertical;min-height:80px}
.fr{display:grid;grid-template-columns:1fr 1fr;gap:1rem}
.rg{display:flex;gap:1rem}
.rc{flex:1;border:2px solid var(--border);border-radius:var(--r12);padding:1rem;cursor:pointer;transition:all .2s;text-align:center}
.rc.on{border-color:var(--teal);background:var(--teal3)}
.rc input{display:none}
.ri{font-size:1.5rem;margin-bottom:.4rem}.rl{font-size:.88rem;font-weight:600}
.er{color:var(--red);font-size:.82rem;margin-top:.5rem}

/* DASHBOARD */
.dw{max-width:1280px;margin:0 auto;padding:2rem clamp(1.5rem,4vw,3rem)}
.dh{display:flex;justify-content:space-between;align-items:center;margin-bottom:2rem;flex-wrap:wrap;gap:1rem}
.dt2{font-family:var(--D);font-size:1.75rem;font-weight:800}
.ds2{color:var(--muted);font-size:.88rem;margin-top:.25rem}
.tabs{display:flex;gap:.25rem;background:#EDF2F7;border-radius:var(--r12);padding:4px;width:fit-content;margin-bottom:1.5rem}
.tab{padding:8px 18px;border-radius:var(--r8);cursor:pointer;font-size:.875rem;font-weight:500;background:none;border:none;color:var(--muted);font-family:var(--B);transition:all .2s}
.tab.on{background:var(--w);color:var(--ink);font-weight:700;box-shadow:0 1px 3px rgba(0,0,0,.08)}
.stg{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:1rem;margin-bottom:2rem}
.stc{background:var(--w);border-radius:var(--r16);padding:1.25rem;border:1px solid var(--border)}
.sv{font-family:var(--D);font-size:2rem;font-weight:800;color:var(--teal)}
.sl2{color:var(--muted);font-size:.78rem;margin-top:3px}
.alist{display:flex;flex-direction:column;gap:.75rem}
.acard{background:var(--w);border-radius:var(--r16);padding:1.25rem 1.5rem;border:1px solid var(--border);display:flex;justify-content:space-between;align-items:center;gap:1rem;flex-wrap:wrap;border-left:4px solid var(--teal);transition:box-shadow .2s}
.acard:hover{box-shadow:var(--sd)}
.acard.cc{border-left-color:var(--muted);opacity:.7}
.acard.pc{border-left-color:var(--border)}
.an{font-weight:700;font-size:.93rem}.ab{color:var(--muted);font-size:.8rem;margin-top:2px}
.badge{padding:4px 12px;border-radius:99px;font-size:.74rem;font-weight:700}
.bco{background:var(--green2);color:#065F46}.bca{background:var(--red2);color:#991B1B}

/* EMPTY */
.empty{display:flex;flex-direction:column;align-items:center;justify-content:center;padding:4rem 2rem;color:var(--muted);text-align:center;gap:1rem}
.ei{width:64px;height:64px;border-radius:50%;background:#EDF2F7;display:flex;align-items:center;justify-content:center;font-size:1.8rem}
.empty h3{font-family:var(--D);color:var(--slate);font-size:1.1rem}

/* TOAST */
.toast{position:fixed;bottom:2rem;right:2rem;z-index:999;display:flex;align-items:center;gap:.75rem;padding:14px 20px;border-radius:var(--r16);background:var(--ink);color:#fff;box-shadow:var(--sdl);font-size:.9rem;font-weight:500;animation:ti .35s cubic-bezier(.34,1.56,.64,1);max-width:320px}
@keyframes ti{from{opacity:0;transform:translateY(20px) scale(.9)}to{opacity:1;transform:translateY(0) scale(1)}}
.tck{width:24px;height:24px;border-radius:50%;background:var(--teal);display:flex;align-items:center;justify-content:center;font-size:.8rem;flex-shrink:0}

/* FOOTER */
.ft{background:var(--ink);color:rgba(255,255,255,.5);padding:2.5rem clamp(1.5rem,4vw,3rem);display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:1rem;font-size:.82rem}
.flogo{font-family:var(--D);font-weight:900;font-size:1.2rem;color:#fff;cursor:pointer;border:none;background:none}
.flinks{display:flex;gap:1.5rem}
.fl{color:rgba(255,255,255,.4);cursor:pointer;transition:color .2s;border:none;background:none;font-family:var(--B);font-size:.82rem}
.fl:hover{color:rgba(255,255,255,.8)}

/* USER */
.uchip{display:flex;align-items:center;gap:.6rem;background:#EDF2F7;border-radius:99px;padding:4px 12px 4px 4px;cursor:pointer;transition:background .2s;border:none}
.uchip:hover{background:var(--border)}
.uav{width:32px;height:32px;border-radius:50%;background:var(--teal);display:flex;align-items:center;justify-content:center;font-weight:800;font-size:.74rem;color:#fff;font-family:var(--D)}
.un{font-size:.82rem;font-weight:600}

/* PROFILE PAGE */
.pp{max-width:900px;margin:0 auto;padding:2rem clamp(1.5rem,4vw,3rem)}
.back{display:inline-flex;align-items:center;gap:.5rem;color:var(--slate);font-size:.88rem;font-weight:600;cursor:pointer;background:none;border:none;font-family:var(--B);padding:0;margin-bottom:1.5rem;transition:color .2s}
.back:hover{color:var(--ink)}
.ph{background:linear-gradient(135deg,var(--ink) 0%,var(--ink2) 100%);border-radius:20px;padding:1.5rem;color:#fff;display:flex;gap:1.25rem;align-items:center;margin-bottom:1.5rem;flex-wrap:wrap}
.pav{width:72px;height:72px;border-radius:var(--r16);flex-shrink:0;display:flex;align-items:center;justify-content:center;font-family:var(--D);font-weight:800;font-size:1.4rem;color:#fff}
.pn{font-family:var(--D);font-size:1.35rem;font-weight:700}
.ps2{opacity:.7;font-size:.88rem;margin-top:3px}
.pg{display:grid;grid-template-columns:1fr 1fr;gap:1.5rem}
.ml{font-size:.74rem;font-weight:700;text-transform:uppercase;letter-spacing:.8px;color:var(--muted);margin-bottom:.5rem}

/* LOADING */
.loading{min-height:100vh;display:flex;align-items:center;justify-content:center;background:var(--ink);color:#fff;font-family:var(--D);font-size:2rem;font-weight:900;letter-spacing:-.5px;flex-direction:column;gap:1rem}
.loading-dot{width:8px;height:8px;border-radius:50%;background:var(--teal);animation:pu 1s infinite}

@media(max-width:768px){
  .hero{grid-template-columns:1fr;min-height:auto;padding:3rem 1.5rem}
  .hvis{display:none}
  .nmid{display:none}
  .nav{padding:0 1rem}
  .sec{padding:48px 1rem}
  .how{padding:48px 1rem}
  .howg{grid-template-columns:1fr}
  .pg{grid-template-columns:1fr}
  .fr{grid-template-columns:1fr}
  .hstats{gap:1.25rem}
}
`;

function Map({ lat, lng }: { lat: number; lng: number }) {
  const bbox = `${lng-.012},${lat-.008},${lng+.012},${lat+.008}`;
  return (
    <div className="mapw">
      <iframe className="mapif" src={`https://www.openstreetmap.org/export/embed.html?bbox=${bbox}&layer=mapnik&marker=${lat},${lng}`} title="map" loading="lazy" />
    </div>
  );
}

export default function App() {
  const [lang, setLang] = useState<Lang>("fr");
  const [langReady, setLangReady] = useState(false);
  const t = T[lang];

  // Detect language from IP on first load
  useEffect(() => {
    const saved = ls.get<Lang|null>("mb_lang", null);
    if (saved) { setLang(saved); setLangReady(true); return; }
    detectLang().then(l => { setLang(l); ls.set("mb_lang", l); setLangReady(true); });
  }, []);

  const changeLang = (l: Lang) => { setLang(l); ls.set("mb_lang", l); };

  const [page, setPage] = useState<"home"|"login"|"register"|"dashboard">("home");
  const [user, setUser] = useState<User|null>(()=>ls.get<User|null>("mb_s",null));
  const [appts, setAppts] = useState<Appointment[]>(()=>ls.get<Appointment[]>("mb_a",[]));
  const [toast, setToast] = useState<string|null>(null);
  const [searchQ, setSearchQ] = useState("");
  const [fSpec, setFSpec] = useState(-1);
  const [selDoc, setSelDoc] = useState<Doctor|null>(null);
  const [viewDoc, setViewDoc] = useState<Doctor|null>(null);
  const [bStep, setBStep] = useState(1);
  const [selSlot, setSelSlot] = useState<Slot|null>(null);
  const [reason, setReason] = useState("");
  const [aTab, setATab] = useState<"upcoming"|"past">("upcoming");
  const [dTab, setDTab] = useState<"appts"|"search">("search");

  const showToast = useCallback((m:string)=>{ setToast(m); setTimeout(()=>setToast(null),3500); },[]);
  const saveA = (a:Appointment[])=>{ setAppts(a); ls.set("mb_a",a); };
  const logout = ()=>{ ls.set("mb_s",null); setUser(null); setPage("home"); };
  const goHome = ()=>{ setViewDoc(null); setPage(user?"dashboard":"home"); };

  const fmtD = (d:string) => new Date(d).toLocaleDateString(
    lang==="fr"?"fr-FR":lang==="ro"?"ro-RO":"en-GB",
    {weekday:"long",day:"numeric",month:"long"}
  );

  const docSlots = (id:number) => {
    const m:Record<string,Slot[]>={};
    ALL_SLOTS.filter(s=>s.doctorId===id).forEach(s=>{(m[s.date]=m[s.date]||[]).push(s);});
    return m;
  };

  const filtered = DOCTORS.filter(d=>{
    const q=searchQ.toLowerCase();
    const sp=fSpec>=0?d.specialty===fSpec:true;
    return sp&&(!q||`${d.firstName} ${d.lastName} ${t.specs[d.specialty]} ${d.city}`.toLowerCase().includes(q));
  });

  if (!langReady) return (
    <div className="loading">
      <style>{CSS}</style>
      MediBook<span className="loading-dot"/>
    </div>
  );

  // ── DOC CARD ─────────────────────────────────────────────────
  const DocCard = ({doc,onView,onBook}:{doc:Doctor;onView:()=>void;onBook:()=>void}) => {
    const first = ALL_SLOTS.find(s=>s.doctorId===doc.id);
    return (
      <div className="dc" onClick={onView}>
        <div className="dct">
          <div className="da" style={{background:doc.color}}>{doc.avatar}</div>
          <div style={{flex:1}}>
            <div className="dn">Dr. {doc.firstName} {doc.lastName}</div>
            <div className="dsp">{t.specs[doc.specialty]}</div>
            <div className="dci">📍 {doc.city}</div>
          </div>
        </div>
        <div className="dtags">
          {doc.available?<span className="tag ta">● {t.avail}</span>:<span className="tag tn">○ Indisponible</span>}
          {doc.teleConsult&&<span className="tag tt">📹 {t.tele}</span>}
        </div>
        <div className="dmeta">
          <div>
            <div style={{display:"flex",alignItems:"center",gap:".4rem"}}>
              <span className="dstars">{"★".repeat(Math.floor(doc.rating))}{"☆".repeat(5-Math.floor(doc.rating))}</span>
              <span className="dr2">{doc.rating}</span>
              <span className="dc3">({doc.reviews} {t.reviews})</span>
            </div>
            <div className="dex">{doc.experience} {t.exp}</div>
          </div>
          <div className="dpr"><strong>{doc.price}€</strong> / consult.</div>
        </div>
        <div className="dfoot">
          <div className="dnx">{first?<>Prochain : <strong>{first.time}</strong></>:<span>{t.noSlots}</span>}</div>
          <button className="btn bsm bp" onClick={e=>{e.stopPropagation();onBook();}}>{t.book} →</button>
        </div>
      </div>
    );
  };

  // ── BOOKING MODAL ────────────────────────────────────────────
  const Modal = () => {
    if (!selDoc) return null;
    const bd = docSlots(selDoc.id);
    const close = () => { setSelDoc(null); setBStep(1); setSelSlot(null); };
    const doBook = () => {
      if (!selSlot||!user) return;
      const a:Appointment = {
        id:Date.now(),doctorId:selDoc.id,patientId:user.id,
        patientName:`${user.firstName} ${user.lastName}`,
        doctorName:`Dr. ${selDoc.firstName} ${selDoc.lastName}`,
        specialty:t.specs[selDoc.specialty],address:selDoc.address,
        date:selSlot.date,time:selSlot.time,reason,status:"confirmed"
      };
      saveA([...appts,a]); close(); setReason(""); showToast(t.booked);
    };
    return (
      <div className="ov" onClick={e=>e.target===e.currentTarget&&close()}>
        <div className="modal">
          <div className="mhandle"/>
          <div className="mh">
            <div className="mt2">{bStep===1?`${t.bookWith} Dr. ${selDoc.lastName}`:t.confStep}</div>
            <button className="mx" onClick={close}>×</button>
          </div>
          <div className="mb2">
            {bStep===1?(
              <>
                <div style={{display:"flex",gap:"1rem",alignItems:"center",padding:"1rem",background:"var(--bg)",borderRadius:"var(--r16)",marginBottom:"1.5rem",border:"1px solid var(--border)"}}>
                  <div className="da" style={{background:selDoc.color,width:48,height:48,fontSize:".9rem",borderRadius:"var(--r12)"}}>{selDoc.avatar}</div>
                  <div style={{flex:1}}>
                    <div style={{fontWeight:700}}>Dr. {selDoc.firstName} {selDoc.lastName}</div>
                    <div style={{color:"var(--teal)",fontSize:".82rem",fontWeight:600}}>{t.specs[selDoc.specialty]}</div>
                    <div style={{color:"var(--muted)",fontSize:".79rem",marginTop:2}}>🏥 {selDoc.address}</div>
                  </div>
                  <div style={{textAlign:"right"}}>
                    <div style={{fontFamily:"var(--D)",fontSize:"1.2rem",fontWeight:800}}>{selDoc.price}€</div>
                    <div style={{color:"var(--muted)",fontSize:".74rem"}}>consultation</div>
                  </div>
                </div>
                <Map lat={selDoc.lat} lng={selDoc.lng}/>
                <div style={{fontSize:".79rem",color:"var(--muted)",margin:".5rem 0 1.5rem"}}>📍 {selDoc.address}</div>
                <div className="slsec">
                  <div className="slt">{t.slots}</div>
                  {Object.keys(bd).length===0
                    ?<div className="empty" style={{padding:"2rem"}}><div className="ei">📅</div><p>{t.noSlots}</p></div>
                    :Object.entries(bd).map(([date,slots])=>(
                      <div key={date} className="sld">
                        <div className="sldl">{fmtD(date)}</div>
                        <div className="slrow">
                          {slots.map(s=><button key={s.id} className={`slot${selSlot?.id===s.id?" on":""}`} onClick={()=>setSelSlot(s)}>{s.time}</button>)}
                        </div>
                      </div>
                    ))
                  }
                </div>
              </>
            ):(
              <>
                <div className="cfc">
                  <div style={{fontSize:"2rem"}}>✓</div>
                  <div>
                    <div className="cfn">Dr. {selDoc.firstName} {selDoc.lastName}</div>
                    <div className="cfs">{selSlot&&fmtD(selSlot.date)} {t.at} {selSlot?.time}</div>
                  </div>
                </div>
                <div className="icrd" style={{marginBottom:"1.5rem"}}>
                  <div className="ict">{t.cabinet}</div>
                  <div className="ir"><span>📍</span><span>{selDoc.address}</span></div>
                  <div className="ir"><span>💶</span><span>{selDoc.price}€ — {selDoc.teleConsult?t.tele+" disponible":"Cabinet uniquement"}</span></div>
                </div>
                <div className="fg">
                  <label className="lbl">{t.reason}</label>
                  <textarea className="tex" placeholder={t.reasonPh} value={reason} onChange={e=>setReason(e.target.value)}/>
                </div>
              </>
            )}
          </div>
          <div className="mf">
            {bStep===2&&<button className="btn bg" onClick={()=>setBStep(1)}>← {lang==="fr"?"Retour":lang==="ro"?"Înapoi":"Back"}</button>}
            <button className="btn bp" disabled={!selSlot} onClick={()=>bStep===1?setBStep(2):doBook()}>
              {bStep===1?(selSlot?`${t.book} — ${selSlot.time}`:t.selSlot):`✓ ${t.confirm}`}
            </button>
          </div>
        </div>
      </div>
    );
  };

  // ── DOCTOR PROFILE ───────────────────────────────────────────
  const Profile = () => {
    if (!viewDoc) return null;
    const d = viewDoc;
    return (
      <div className="pp">
        <button className="back" onClick={()=>setViewDoc(null)}>{t.back}</button>
        <div className="ph">
          <div className="pav" style={{background:d.color}}>{d.avatar}</div>
          <div style={{flex:1,minWidth:200}}>
            <div className="pn">Dr. {d.firstName} {d.lastName}</div>
            <div className="ps2">{t.specs[d.specialty]} · {d.city}</div>
            <div style={{display:"flex",gap:"1rem",marginTop:"1rem",flexWrap:"wrap"}}>
              {d.available&&<span style={{background:"rgba(16,185,129,.2)",color:"#6EE7B7",padding:"4px 12px",borderRadius:99,fontSize:".78rem",fontWeight:700}}>● {t.avail}</span>}
              {d.teleConsult&&<span style={{background:"rgba(139,92,246,.2)",color:"#C4B5FD",padding:"4px 12px",borderRadius:99,fontSize:".78rem",fontWeight:700}}>📹 {t.tele}</span>}
            </div>
          </div>
          <div style={{textAlign:"right",flexShrink:0}}>
            <div style={{fontFamily:"var(--D)",fontSize:"2rem",fontWeight:900}}>{d.price}€</div>
            <div style={{opacity:.6,fontSize:".82rem"}}>/consultation</div>
          </div>
        </div>
        <div className="pg">
          <div>
            <div className="icrd" style={{marginBottom:"1rem"}}>
              <div className="ict">Informations</div>
              <div className="ir"><span>⭐</span><span><strong>{d.rating}</strong> — {d.reviews} {t.reviews}</span></div>
              <div className="ir"><span>🩺</span><span>{d.experience} {t.exp}</span></div>
              <div className="ir"><span>💶</span><span>{d.price}€ / consultation</span></div>
              <div className="ir"><span>📹</span><span>{d.teleConsult?t.tele+" disponible":"Cabinet uniquement"}</span></div>
            </div>
            <div className="icrd">
              <div className="ict">{t.cabinet}</div>
              <div className="ir"><span>📍</span><span>{d.address}</span></div>
            </div>
          </div>
          <div>
            <div className="ml">Localisation</div>
            <Map lat={d.lat} lng={d.lng}/>
          </div>
        </div>
        <div style={{background:"var(--w)",borderRadius:"var(--r24)",padding:"1.5rem",marginTop:"1.5rem",border:"1px solid var(--border)"}}>
          <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"1.25rem",flexWrap:"wrap",gap:"1rem"}}>
            <div>
              <div style={{fontFamily:"var(--D)",fontWeight:700,fontSize:"1.1rem"}}>{t.slots}</div>
              <div style={{color:"var(--muted)",fontSize:".82rem",marginTop:2}}>7 {lang==="fr"?"prochains jours":lang==="ro"?"zile viitoare":"next days"}</div>
            </div>
            <button className="btn bp" onClick={()=>{if(!user){setPage("login");return;}setSelDoc(d);}}>{t.book} →</button>
          </div>
          {Object.entries(docSlots(d.id)).slice(0,3).map(([date,slots])=>(
            <div key={date} className="sld">
              <div className="sldl">{fmtD(date)}</div>
              <div className="slrow">
                {slots.map(s=>(
                  <button key={s.id} className="slot" onClick={()=>{if(!user){setPage("login");return;}setSelDoc(d);setSelSlot(s);setBStep(2);}}>
                    {s.time}
                  </button>
                ))}
              </div>
            </div>
          ))}
        </div>
      </div>
    );
  };

  // ── HOME ─────────────────────────────────────────────────────
  const Home = () => (
    <>
      <section className="hero">
        <div className="hbg"/><div className="hgrid"/>
        <div className="hcnt">
          <div className="hbadge"><span className="hbd"/>24h/24 · 7j/7</div>
          <h1 className="hh1">{t.heroTitle}<br/><em>{t.heroTitle2}</em></h1>
          <p className="hsub">{t.heroSub}</p>
          <div className="hcta">
            <button className="btn bt blg" onClick={()=>document.getElementById("docs")?.scrollIntoView({behavior:"smooth"})}>{t.searchBtn} →</button>
            <button className="btn bo blg" onClick={()=>setPage(user?"dashboard":"register")}>
              {user?t.dash:t.register}
            </button>
          </div>
          <div className="hstats">
            {[["50k+",t.sp],["200+",t.sd],["12k",t.sa],["4.8★",t.sr]].map(([v,l])=>(
              <div key={l}><div className="hsv">{v}</div><div className="hsl">{l}</div></div>
            ))}
          </div>
        </div>
        <div className="hvis">
          {DOCTORS.slice(0,3).map((d,i)=>{
            const sl=ALL_SLOTS.filter(s=>s.doctorId===d.id).slice(0,3);
            return (
              <div key={d.id} className="hcard" style={{animationDelay:`${-i*2}s`}}>
                <div className="hcrow">
                  <div className="havt" style={{background:d.color}}>{d.avatar}</div>
                  <div><div className="hcn">Dr. {d.firstName} {d.lastName}</div><div className="hcs">{t.specs[d.specialty]} · {d.city}</div></div>
                </div>
                {sl.length>0&&<div className="hslots">{sl.map(s=><span key={s.id} className="hsl2">{s.time}</span>)}</div>}
              </div>
            );
          })}
        </div>
      </section>

      <div className="sw">
        <div className="sb">
          <div className="sf">
            <span>🔍</span>
            <input placeholder={t.searchLabel} value={searchQ} onChange={e=>setSearchQ(e.target.value)}/>
          </div>
          <div className="ssw">
            <span>📍</span>
            <select className="ss" value={fSpec} onChange={e=>setFSpec(parseInt(e.target.value))}>
              <option value={-1}>{t.allSpec}</option>
              {t.specs.map((s,i)=><option key={i} value={i}>{s}</option>)}
            </select>
          </div>
          <div className="sbw"><button className="btn bp">{t.searchBtn}</button></div>
        </div>
        <div className="chips">
          {t.specs.slice(0,6).map((s,i)=>(
            <button key={i} className="chip" onClick={()=>setFSpec(i===fSpec?-1:i)}
              style={i===fSpec?{borderColor:"var(--teal)",color:"var(--teal)",background:"var(--teal3)"}:{}}>{s}</button>
          ))}
        </div>
      </div>

      <div className="how">
        <div style={{fontFamily:"var(--D)",fontSize:"clamp(1.5rem,3vw,2rem)",fontWeight:700,color:"#fff",marginBottom:".5rem"}}>{t.howTitle}</div>
        <div style={{color:"rgba(255,255,255,.5)",fontSize:".88rem"}}>Simple, rapide, fiable.</div>
        <div className="howg">
          {[["🔍",t.s1t,t.s1d,"01"],["📅",t.s2t,t.s2d,"02"],["✓",t.s3t,t.s3d,"03"]].map(([icon,title,desc,n])=>(
            <div key={n} className="howc">
              <div className="hown">{n}</div>
              <div className="howi">{icon}</div>
              <div className="howt">{title as string}</div>
              <div className="howd">{desc as string}</div>
            </div>
          ))}
        </div>
      </div>

      <section className="sec" id="docs">
        <div className="sech">
          <div>
            <div className="sect">{t.doctors}</div>
            <div className="secs">{filtered.length} médecin{filtered.length>1?"s":""} trouvé{filtered.length>1?"s":""}</div>
          </div>
        </div>
        {filtered.length===0
          ?<div className="empty"><div className="ei">🔍</div><h3>Aucun résultat</h3></div>
          :<div className="dgrid">{filtered.map(d=>(
            <DocCard key={d.id} doc={d}
              onView={()=>{setViewDoc(d);window.scrollTo({top:0,behavior:"smooth"});}}
              onBook={()=>{if(!user){setPage("login");return;}setSelDoc(d);}}
            />
          ))}</div>
        }
      </section>

      <footer className="ft">
        <div>
          <button className="flogo" onClick={goHome}>MediBook</button>
          <div style={{marginTop:".4rem"}}>{t.footerTag}</div>
        </div>
        <div className="flinks">
          {[t.about,t.contact,t.legal].map(l=><button key={l} className="fl">{l}</button>)}
        </div>
        <div>© 2025 MediBook</div>
      </footer>
    </>
  );

  // ── LOGIN ────────────────────────────────────────────────────
  const Login = () => {
    const [f,setF]=useState({email:"",password:""});
    const [err,setErr]=useState("");
    const go=()=>{
      const users=ls.get<User[]>("mb_u",[]);
      const u=users.find(u=>u.email===f.email&&u.password===f.password);
      if(!u){setErr(lang==="fr"?"Identifiants incorrects":lang==="ro"?"Date incorecte":"Invalid credentials");return;}
      ls.set("mb_s",u);setUser(u);setPage("dashboard");
    };
    return (
      <div className="aw">
        <div className="ac">
          <button className="alogo" onClick={()=>setPage("home")}>MediBook</button>
          <div className="at">{t.login}</div>
          <div className="as2">{t.welcome} 👋</div>
          <div className="fg"><label className="lbl">{t.email}</label><input className="inp" type="email" value={f.email} onChange={e=>setF({...f,email:e.target.value})}/></div>
          <div className="fg"><label className="lbl">{t.password}</label><input className="inp" type="password" value={f.password} onChange={e=>setF({...f,password:e.target.value})} onKeyDown={e=>e.key==="Enter"&&go()}/></div>
          {err&&<div className="er">{err}</div>}
          <button className="btn bp bw" style={{marginTop:"1rem"}} onClick={go}>{t.login}</button>
          <div className="atg">{t.noAcc} <a onClick={()=>setPage("register")}>{t.register}</a></div>
        </div>
      </div>
    );
  };

  // ── REGISTER ────────────────────────────────────────────────
  const Register = () => {
    const [f,setF]=useState({firstName:"",lastName:"",email:"",password:"",phone:"",role:"patient" as Role,specialty:0,city:"",address:""});
    const [err,setErr]=useState("");
    const go=()=>{
      if(!f.email||!f.password||!f.firstName||!f.lastName){setErr(lang==="fr"?"Champs requis manquants":"Required fields missing");return;}
      const users=ls.get<User[]>("mb_u",[]);
      if(users.find(u=>u.email===f.email)){setErr(lang==="fr"?"Email déjà utilisé":"Email already in use");return;}
      const nu:User={...f,id:Date.now()};
      ls.set("mb_u",[...users,nu]);ls.set("mb_s",nu);setUser(nu);setPage("dashboard");
    };
    return (
      <div className="aw">
        <div className="ac">
          <button className="alogo" onClick={()=>setPage("home")}>MediBook</button>
          <div className="at">{t.register}</div>
          <div className="as2">Rejoignez MediBook ✨</div>
          <div className="fg">
            <label className="lbl">{t.accType}</label>
            <div className="rg">
              {([["patient","🤒",t.patient],["doctor","🩺",t.doctor]] as [Role,string,string][]).map(([v,icon,lbl])=>(
                <label key={v} className={`rc${f.role===v?" on":""}`} onClick={()=>setF({...f,role:v})}>
                  <input type="radio" checked={f.role===v} onChange={()=>{}}/><div className="ri">{icon}</div><div className="rl">{lbl}</div>
                </label>
              ))}
            </div>
          </div>
          <div className="fr">
            <div className="fg"><label className="lbl">{t.firstName} *</label><input className="inp" value={f.firstName} onChange={e=>setF({...f,firstName:e.target.value})}/></div>
            <div className="fg"><label className="lbl">{t.lastName} *</label><input className="inp" value={f.lastName} onChange={e=>setF({...f,lastName:e.target.value})}/></div>
          </div>
          <div className="fg"><label className="lbl">{t.email} *</label><input className="inp" type="email" value={f.email} onChange={e=>setF({...f,email:e.target.value})}/></div>
          <div className="fg"><label className="lbl">{t.password} *</label><input className="inp" type="password" value={f.password} onChange={e=>setF({...f,password:e.target.value})}/></div>
          <div className="fr">
            <div className="fg"><label className="lbl">{t.phone}</label><input className="inp" value={f.phone} onChange={e=>setF({...f,phone:e.target.value})}/></div>
            <div className="fg"><label className="lbl">{t.city}</label><input className="inp" value={f.city} onChange={e=>setF({...f,city:e.target.value})}/></div>
          </div>
          {f.role==="doctor"&&<>
            <div className="fg"><label className="lbl">{t.spec}</label>
              <select className="sel" value={f.specialty} onChange={e=>setF({...f,specialty:parseInt(e.target.value)})}>
                {t.specs.map((s,i)=><option key={i} value={i}>{s}</option>)}
              </select>
            </div>
            <div className="fg"><label className="lbl">{t.address}</label><input className="inp" placeholder={t.addrPh} value={f.address} onChange={e=>setF({...f,address:e.target.value})}/></div>
          </>}
          {err&&<div className="er">{err}</div>}
          <button className="btn bp bw" style={{marginTop:"1rem"}} onClick={go}>{t.register}</button>
          <div className="atg">{t.hasAcc} <a onClick={()=>setPage("login")}>{t.login}</a></div>
        </div>
      </div>
    );
  };

  // ── PATIENT DASH ─────────────────────────────────────────────
  const PatDash = () => {
    if(!user) return null;
    const now=new Date().toISOString().split("T")[0];
    const mine=appts.filter(a=>a.patientId===user.id);
    const up=mine.filter(a=>a.date>=now&&a.status!=="cancelled");
    const pa=mine.filter(a=>a.date<now||a.status==="cancelled");
    const shown=aTab==="upcoming"?up:pa;
    const cancelA=(id:number)=>{saveA(appts.map(a=>a.id===id?{...a,status:"cancelled" as Status}:a));showToast(lang==="fr"?"RDV annulé":lang==="ro"?"Programare anulată":"Appointment cancelled");};
    if(viewDoc) return <Profile/>;
    return (
      <div className="dw">
        <div className="dh">
          <div><div className="dt2">{lang==="fr"?"Bonjour":lang==="ro"?"Bună ziua":"Hello"}, {user.firstName} 👋</div>
          <div className="ds2">{new Date().toLocaleDateString(lang==="fr"?"fr-FR":lang==="ro"?"ro-RO":"en-GB",{weekday:"long",day:"numeric",month:"long",year:"numeric"})}</div></div>
          <button className="btn bp" onClick={()=>setDTab("search")}>{t.book} +</button>
        </div>
        <div className="tabs">
          {([["appts",t.myAppts],["search",t.findDoc]] as [string,string][]).map(([v,l])=>(
            <button key={v} className={`tab${dTab===v?" on":""}`} onClick={()=>setDTab(v as "appts"|"search")}>{l}</button>
          ))}
        </div>
        {dTab==="appts"&&<>
          <div className="stg">
            <div className="stc"><div className="sv">{up.length}</div><div className="sl2">{t.upcoming}</div></div>
            <div className="stc"><div className="sv">{pa.length}</div><div className="sl2">{t.past}</div></div>
            <div className="stc"><div className="sv">{up[0]?.time||"–"}</div><div className="sl2">{t.nextAppt}</div></div>
          </div>
          <div className="tabs" style={{marginBottom:"1rem"}}>
            <button className={`tab${aTab==="upcoming"?" on":""}`} onClick={()=>setATab("upcoming")}>{t.upcoming} ({up.length})</button>
            <button className={`tab${aTab==="past"?" on":""}`} onClick={()=>setATab("past")}>{t.past} ({pa.length})</button>
          </div>
          {shown.length===0
            ?<div className="empty"><div className="ei">📋</div><h3>{t.noAppts}</h3><button className="btn bp" style={{marginTop:".5rem"}} onClick={()=>setDTab("search")}>{t.book}</button></div>
            :<div className="alist">{shown.map(a=>(
              <div key={a.id} className={`acard${a.status==="cancelled"?" cc":a.date<now?" pc":""}`}>
                <div>
                  <div className="an">{a.doctorName}</div>
                  <div className="ab">{a.specialty} · {fmtD(a.date)} {t.at} {a.time}</div>
                  {a.address&&<div className="ab">📍 {a.address}</div>}
                  {a.reason&&<div className="ab" style={{fontStyle:"italic"}}>"{a.reason}"</div>}
                </div>
                <div style={{display:"flex",alignItems:"center",gap:".5rem",flexShrink:0}}>
                  <span className={`badge b${a.status==="confirmed"?"co":"ca"}`}>{t[a.status]}</span>
                  {a.status==="confirmed"&&a.date>=now&&<button className="btn bdan bsm" onClick={()=>cancelA(a.id)}>{t.cancel}</button>}
                </div>
              </div>
            ))}</div>
          }
        </>}
        {dTab==="search"&&<>
          <div style={{display:"flex",gap:"1rem",marginBottom:"1.5rem",flexWrap:"wrap"}}>
            <div style={{flex:1,minWidth:200,position:"relative"}}>
              <span style={{position:"absolute",left:12,top:"50%",transform:"translateY(-50%)",color:"var(--muted)"}}>🔍</span>
              <input className="inp" style={{paddingLeft:36}} placeholder={t.searchLabel} value={searchQ} onChange={e=>setSearchQ(e.target.value)}/>
            </div>
            <select className="sel" style={{width:"auto"}} value={fSpec} onChange={e=>setFSpec(parseInt(e.target.value))}>
              <option value={-1}>{t.allSpec}</option>
              {t.specs.map((s,i)=><option key={i} value={i}>{s}</option>)}
            </select>
          </div>
          <div className="dgrid">{filtered.map(d=>(
            <DocCard key={d.id} doc={d}
              onView={()=>{setViewDoc(d);window.scrollTo({top:0,behavior:"smooth"});}}
              onBook={()=>setSelDoc(d)}
            />
          ))}</div>
        </>}
      </div>
    );
  };

  // ── DOCTOR DASH ──────────────────────────────────────────────
  const DocDash = () => {
    if(!user) return null;
    const now=new Date().toISOString().split("T")[0];
    const mid=DOCTORS.find(d=>d.lastName===user.lastName)?.id;
    const mine=appts.filter(a=>a.doctorId===mid);
    const up=mine.filter(a=>a.date>=now&&a.status!=="cancelled");
    const pa=mine.filter(a=>a.date<now||a.status==="cancelled");
    const shown=aTab==="upcoming"?up:pa;
    return (
      <div className="dw">
        <div className="dh">
          <div><div className="dt2">Dr. {user.firstName} {user.lastName}</div><div className="ds2">{t.specs[user.specialty]||""}</div></div>
        </div>
        <div className="stg">
          <div className="stc"><div className="sv">{up.length}</div><div className="sl2">{t.upcoming}</div></div>
          <div className="stc"><div className="sv">{new Set(mine.map(a=>a.patientId)).size}</div><div className="sl2">{t.totalPat}</div></div>
          <div className="stc"><div className="sv">{mine.filter(a=>a.date===now).length}</div><div className="sl2">{t.today}</div></div>
          <div className="stc"><div className="sv">{up[0]?.time||"–"}</div><div className="sl2">{t.nextAppt}</div></div>
        </div>
        <div className="tabs">
          <button className={`tab${aTab==="upcoming"?" on":""}`} onClick={()=>setATab("upcoming")}>{t.upcoming} ({up.length})</button>
          <button className={`tab${aTab==="past"?" on":""}`} onClick={()=>setATab("past")}>{t.past} ({pa.length})</button>
        </div>
        {shown.length===0
          ?<div className="empty"><div className="ei">🩺</div><h3>{t.noAppts}</h3></div>
          :<div className="alist">{shown.map(a=>(
            <div key={a.id} className={`acard${a.status==="cancelled"?" cc":a.date<now?" pc":""}`}>
              <div><div className="an">{a.patientName}</div><div className="ab">{fmtD(a.date)} {t.at} {a.time}</div>{a.reason&&<div className="ab" style={{fontStyle:"italic"}}>"{a.reason}"</div>}</div>
              <span className={`badge b${a.status==="confirmed"?"co":"ca"}`}>{t[a.status]}</span>
            </div>
          ))}</div>
        }
      </div>
    );
  };

  // ── RENDER ────────────────────────────────────────────────────
  const content = () => {
    if(page==="login") return <Login/>;
    if(page==="register") return <Register/>;
    if(page==="dashboard"&&user) return user.role==="doctor"?<DocDash/>:<PatDash/>;
    if(viewDoc) return <Profile/>;
    return <Home/>;
  };

  return (
    <>
      <style>{CSS}</style>
      <div style={{minHeight:"100vh",display:"flex",flexDirection:"column"}}>
        <nav className="nav">
          <button className="nlogo" onClick={goHome}>MediBook<span className="ndot"/></button>
          {!["login","register"].includes(page)&&(
            <div className="nmid">
              <button className="nl" onClick={()=>{setViewDoc(null);setPage("home");}}>
                {lang==="fr"?"Accueil":lang==="ro"?"Acasă":"Home"}
              </button>
              <button className="nl" onClick={()=>{setPage("home");setTimeout(()=>document.getElementById("docs")?.scrollIntoView({behavior:"smooth"}),100);}}>
                {lang==="fr"?"Médecins":lang==="ro"?"Medici":"Doctors"}
              </button>
              {user&&<button className="nl" onClick={()=>{setViewDoc(null);setPage("dashboard");}}>{t.dash}</button>}
            </div>
          )}
          <div className="nr">
            <div className="lp">
              {(["fr","en","ro"] as Lang[]).map(l=>(
                <button key={l} className={`lb${lang===l?" on":""}`} onClick={()=>changeLang(l)}>{T[l].langs[l]}</button>
              ))}
            </div>
            {user?(
              <div style={{display:"flex",alignItems:"center",gap:".75rem"}}>
                <button className="uchip" onClick={()=>{setViewDoc(null);setPage("dashboard");}}>
                  <div className="uav">{user.firstName?.[0]}{user.lastName?.[0]}</div>
                  <span className="un">{user.firstName}</span>
                </button>
                <button className="btn bg bsm" onClick={logout}>{t.logout}</button>
              </div>
            ):(
              <>
                <button className="btn bg bsm" onClick={()=>setPage("login")}>{t.login}</button>
                <button className="btn bp bsm" onClick={()=>setPage("register")}>{t.register}</button>
              </>
            )}
          </div>
        </nav>
        <main style={{flex:1}}>{content()}</main>
        {selDoc&&user&&<Modal/>}
        {toast&&<div className="toast"><div className="tck">✓</div>{toast}</div>}
      </div>
    </>
  );
}

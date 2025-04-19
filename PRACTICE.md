Lorem ipsum dolor sit amet, consectetur adipiscing elit. 
non convallis odio lacus a justo. Sed blandit ligula vitae purus pharetra, 
non conv.allis odio lacus a justo. Sed blandit ligula vitae purus pharetra, 
non convallis odio lacus a justo. Sed blandit ligula vitae purus pharetra, 
vel feugiat erat euismod.

# TODO: Replace this line with a motivational quote.
# FIXME: Remove duplicate lorem entries above.

# TODO: This is such a motivational fucking bullshit
# FIXME: Remove duplicate lorem entries above.

# ####################################################################

import { getConnection, sql } from "../database/connection.js";
import url from "url";
import { parseLocalDate } from "../public/js/date.js";
import { sendMail } from "../public/js/sender.js";  

export const getVacaciones = async (req, res) => {
  try {
  
    const pool = await getConnection();

    const result = await pool
      .request()
      .query("SELECT * FROM Vista_SolicitudesVacaciones");

    var vacaciones = result.recordset;

    const mensaje = req.query.msg;

    return res.render("listarVacaciones", 
      {
        msg: mensaje,
        vacaciones: vacaciones,
        parseLocalDate,
        data: {},
        userRole: req.session.user.id_rol,
      }
    );

  } catch (error) {

    return res.status(500).send(error.message);

  }

};

export const getMisVacaciones = async (req, res) => {
  try {

    const pool = await getConnection();

    const result = await pool
      .request()
      .query("SELECT * FROM Vista_VacacionesModificadas");

    var vacaciones = result.recordset;

    const mensaje = req.query.msg;

    return res.render("misVacaciones", 
      {
        msg: mensaje,
        vacaciones: vacaciones,
        usuario : req.session.user,
        parseLocalDate,
        data: {},
        userRole: req.session.user.id_rol,
      }
    );

  } catch (error) {

    return res.status(500).send(error.message);

  }

};

export const getSolicitarVacacion = async (req, res) => {
  try {

    const pool = await getConnection();

    const result = await pool
      .request()
      .query("SELECT * FROM Vista_Managers_Recursos_Humanos");

    var usuarios = result.recordset || req.query.msg;

    const mensaje = req.query.msg;

    return res.render("solicitarVacaciones", {
      msg: mensaje,
      usuarios: usuarios,
      data: {},
      userRole: req.session.user.id_rol,
    });

  } catch (error) {

    return res.status(500).send(error.message);

  }
};


export const getGestorVacaciones = async (req, res) => {
  try {

    
    const pool = await getConnection();
    
    const result = await pool
    .request()
    .query("SELECT * FROM Vista_SolicitudesVacaciones");
    
    var vacaciones = result.recordset;
    
    const mensaje = req.query.msg;
    
    console.log(vacaciones)

    return res.render("gestorVacaciones", 
      {
        msg: mensaje,
        vacaciones: vacaciones,
        usuario : req.session.user,
        parseLocalDate,
        data: {},
        userRole: req.session.user.id_rol,
      }
    );

  } catch (error) { 

    return res.status(500).send(error.message);

  }

};

export const postSolicitarVacacion = async (req, res) => {
  try {
    if (!req.body.fecha_inicio || !req.body.fecha_fin || !req.body.id_manager || !req.body.id_rrhh || !req.body.motivo) {
  
      return res.redirect(
        url.format({
          pathname: "/api/vacaciones/solicitar",
          query: {
            msg: "Por favor, ingrese todos los datos solicitados.",
          },
        })
      );
    }

    const pool = await getConnection();

    const result = await pool
      .request()
      .input("id_usuario", sql.Int, req.session.user.id_usuario)
      .input("id_manager", sql.Int, req.body.id_manager)
      .input("id_rrhh", sql.Int, req.body.id_rrhh)
      .input("fecha_inicio", sql.Date, req.body.fecha_inicio)
      .input("fecha_fin", sql.Date, req.body.fecha_fin)
      .input("motivo", sql.Text, req.body.motivo)
      .execute("SolicitarVacaciones");

    sendMail(req.session.user.correo,"Solicitud de vacación", "Su solicitud de vacación ha sido enviada correctamente, por favor espere la respuesta del gestor de recursos humanos."); 
  
    res.redirect(url.format({
      pathname:"/api/vacaciones/misVacaciones",
      query: {
        msg: "Vacación creada correctamente"
      }
    }));

  } catch (error) {
    return res.status(500).send(error.message);
  }
};

export const postAceptarVacacion = async (req, res) => {
  try {

    if (!req.body.id_vacacion || !req.body.id_usuario || !req.body.motivo || !req.body.correo) {
      return res.redirect(
        url.format({
          pathname: "gestor",
          query: {
            msg: "Por favor, ingrese todos los datos solicitados.",
          },
        })
      );
    }
    
    const pool = await getConnection();
    
    const result = await pool
    .request()
    .input("id_vacacion", sql.Int, req.body.id_vacacion)
    .input("id_usuario", sql.Int, req.body.id_usuario)
    .input("motivo", sql.Text, req.body.motivo)
    .execute("AceptarVacaciones");
 
    sendMail(req.body.correo,"Solicitud de vacación", "Su solicitud de vacación ha sido aceptada correctamente, por favor consulte con su gestor de recursos humanos."); 
     
    return res.redirect(
      url.format({
        pathname: "/api/vacaciones",
        query: {
          msg: "Vacación aceptada correctamente",
        },  
      })
    );
  } catch (error) {
    return res.status(500).send(error.message);
  } 
};

export const postRechazarVacacion = async (req, res) => {
  try {

    console.log(req.body);    

    if (!req.body.id_vacacion || !req.body.id_usuario || !req.body.motivo) {
      return res.redirect(
        url.format({
          pathname: "gestor",
          query: {
            msg: "Por favor, ingrese todos los datos solicitados.",
          },
        })
      );
    }
    
    const pool = await getConnection();
    
    const result = await pool
      .request()
      .input("id_vacacion", sql.Int, req.body.id_vacacion)
      .input("id_usuario", sql.Int, req.body.id_usuario)
      .input("motivo", sql.Text, req.body.motivo)
      .execute("RechazarVacaciones");

    sendMail(req.body.correo,"Solicitud de vacación", "Su solicitud de vacación ha sido rechazada, por favor consulte con su gestor de recursos humanos."); 
  
    return res.redirect(
      url.format({
        pathname: "/api/vacaciones",
        query: {
          msg: "Vacación rechazada correctamente",
        },
      })
    );
  } catch (error) {
    return res.status(500).send(error.message);
  }
};

    );
  } catch (error) {
    return res.status(500).send(error.message);
  }
};


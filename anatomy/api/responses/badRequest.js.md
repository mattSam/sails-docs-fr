# api/responses/badRequest.js
### Objectif

Il s'agit de l'une des réponses serveur par défaut incluses dans un nouveau projet Sails.

Il contient la logique de ce qui devrait se produire lorsque vous souhaitez émettre une réponse http 400. La réponse peut être émise à partir de n'importe quel endroit où vous avez accès à l'objet express `res` en appelant res.badRequest.

N'hésitez pas à modifier ce fichier en fonction de vos besoins. Vous pouvez également créer une réponse personnalisée en utilisant notre générateur `sails-generate-custom-response`.

Pour plus d'informations, reportez-vous à la section Réponse de la documentation de référence.



<docmeta name="displayName" value="badRequest.js">

```
/**
 * 400 (Bad Request) Handler
 *
 * Usage:
 * return res.badRequest();
 * return res.badRequest(err);
 * return res.badRequest(err, view);
 * return res.badRequest(err, redirectTo);
 *
 * e.g.:
 * return res.badRequest(
 *   'Please choose a valid `password` (6-12 characters)',
 *   '/trial/signup'
 * );
 *
 */

module.exports = function badRequest(err, viewOrRedirect) {

  // Get access to `req` & `res`
  var req = this.req;
  var res = this.res;

  // Serve JSON (with optional JSONP support)
  function sendJSON (data) {
    if (!data) {
      return res.send();
    }
    else {
      if (typeof data !== 'object' || data instanceof Error) {
        data = {error: data};
      }
      if ( req.options.jsonp && !req.isSocket ) {
        return res.jsonp(data);
      }
      else return res.json(data);
    }
  }

  // Set status code
  res.status(400);

  // Log error to console
  this.req._sails.log.verbose('Sent 400 ("Bad Request") response');
  if (err) {
    this.req._sails.log.verbose(err);
  }

  // If the user-agent wants JSON, always respond with JSON
  if (req.wantsJSON) {
    return sendJSON(err);
  }

  // Make data more readable for view locals
  var locals;
  if (!err) { locals = {}; }
  else if (typeof err !== 'object'){
    locals = {error: err};
  }
  else {
    var readabilify = function (value) {
      if (sails.util.isArray(value)) {
        return sails.util.map(value, readabilify);
      }
      else if (sails.util.isPlainObject(value)) {
        return sails.util.inspect(value);
      }
      else return value;
    };
    locals = { error: readabilify(err) };
  }

  // Serve HTML view or redirect to specified URL
  if (typeof viewOrRedirect === 'string') {
    if (viewOrRedirect.match(/^(\/|http:\/\/|https:\/\/)/)) {
      return res.redirect(viewOrRedirect);
    }
    else return res.view(viewOrRedirect, locals, function viewReady(viewErr, html) {
      if (viewErr) return sendJSON(err);
      else return res.send(html);
    });
  }
  else return res.view('400', locals, function viewReady(viewErr, html) {
    if (viewErr) return sendJSON(err);
    else return res.send(html);
  });
};


```
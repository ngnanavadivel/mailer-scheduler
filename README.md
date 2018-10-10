# DailyJobScheduler

```java
package com.gnanavad.utils;

import java.text.ParseException;
import java.util.Calendar;
import java.util.Date;
import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.TimeUnit;

public class DailyJobScheduler {

	public static void main(String[] args) {
		Calendar today = Calendar.getInstance();

		// every night at 2am you run your task
		Timer timer = new Timer();
		timer.schedule(new BDayMailerTask("10.10.1980"), today.getTime(),
				TimeUnit.MILLISECONDS.convert(10, TimeUnit.SECONDS)); // period: 10 seconds

	}

}

class BDayMailerTask extends TimerTask {

	private String bday = null;

	public BDayMailerTask(String bday) {
		this.bday = bday;
	}

	@Override
	public void run() {
		Date parsed = null;
		try {
			parsed = BirthdayFinder.parseDate(this.bday);
			System.out.println(BirthdayFinder.isTodayABirthday(parsed) ? "Your birthday today!" : "Not Today!");
		} catch (ParseException e) {
			e.printStackTrace();
		}

	}

}
```

# BirthdayFinder.java

```java
package com.gnanavad.utils;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Arrays;
import java.util.Calendar;
import java.util.Date;
import java.util.List;
import java.util.stream.Collectors;

import com.gnanavad.utils.model.StudentDetails;

public class BirthdayFinder {

	private static final SimpleDateFormat fmt = new SimpleDateFormat("dd.MM.yyyy");

	public static int getDayOfMonth(Date birthday) {
		Calendar instance = Calendar.getInstance();
		instance.setTime(birthday);
		return instance.get(Calendar.DAY_OF_MONTH);
	}

	public static Date parseDate(String birthday) throws ParseException {
		return fmt.parse(birthday);
	}

	public static int getMonth(Date birthday) {
		Calendar instance = Calendar.getInstance();
		instance.setTime(birthday);
		return instance.get(Calendar.MONTH);
	}

	public static boolean isTodayABirthday(Date birthday) {
		int day = getDayOfMonth(birthday);
		int month = getMonth(birthday);
		Date today = new Date();
		int todaysDay = getDayOfMonth(today);
		int todaysMonth = getMonth(today);
		return day == todaysDay && month == todaysMonth;
	}

	public static List<StudentDetails> getTodaysBirthdayBabies(List<StudentDetails> students) {
		return students.stream().filter(e -> {
			try {
				return isTodayABirthday(parseDate(e.getDob()));
			} catch (ParseException e1) {
				e1.printStackTrace();
			}
			return false;
		}).collect(Collectors.toList());
	}
	
	public static void main(String[] args) throws ParseException {
		System.out.println(isTodayABirthday(parseDate("10.10.2018")));
	}
}

```

# MailingUtility.java

```java
package com.gnanavad.utils;

import java.util.Arrays;
import java.util.List;
import java.util.Properties;

import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.AddressException;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

public class MailingUtility {
	public static void sendEmail(String fromEmailId, String fromEmailPassword, List<String> recepients, String subject,
			String body) {

		Properties props = new Properties();
		props.put("mail.smtp.host", "smtp.gmail.com");
		props.put("mail.smtp.socketFactory.port", "465");
		props.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
		props.put("mail.smtp.auth", "true");
		props.put("mail.smtp.port", "465");

		Session session = Session.getInstance(props, new javax.mail.Authenticator() {
			protected PasswordAuthentication getPasswordAuthentication() {
				return new PasswordAuthentication(fromEmailId, fromEmailPassword);
			}
		});

		try {

			InternetAddress[] toAddresses = convertStringToInetAddresses(recepients);

			Message message = new MimeMessage(session);
			message.setFrom(new InternetAddress(fromEmailId));
			message.setRecipients(Message.RecipientType.TO, toAddresses);
			message.setSubject(subject);
			message.setText(body);

			Transport.send(message);

			System.out.println("Done");

		} catch (MessagingException e) {
			throw new RuntimeException(e);
		}
	}

	private static InternetAddress[] convertStringToInetAddresses(List<String> recepients) throws AddressException {
		InternetAddress[] addresses = null;
		if (!recepients.isEmpty()) {
			int size = recepients.size();
			addresses = new InternetAddress[size];
			for (int i = 0; i < size; ++i) {
				InternetAddress[] parsed = InternetAddress.parse(recepients.get(i));
				addresses[i] = (parsed != null && parsed.length > 0) ? parsed[0] : null;
			}
		}
		return addresses;
	}

	public static void main(String[] args) {
		final String fromEmailId = "mail2utillabs@gmail.com";
		final String fromEmailPassword = "Ut1l1ty$";
		List<String> recepients = Arrays
				.asList(new String[] { "nataraj.gnanavadivel@gmail.com", "geetha_mails@yahoo.com" });
		String subject = "Test Mail!!!";
		String body = "Happy Birthday, buddy!";
		MailingUtility.sendEmail(fromEmailId, fromEmailPassword, recepients, subject, body);
	}
}

```

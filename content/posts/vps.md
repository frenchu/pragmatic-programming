+++
date = 2022-04-02T17:34:31+02:00
title = "Affordable cloud solutions for programmers"
description = "VPS comparison"
tags = ["vps", "digitalocean", "linode", "vultr", "upcloud"]
categories = ["devops", "cloud"]
+++

## Introduction

## Results

### CPU performance

{{< chart 150 400 >}}
{
  type: 'bar',
  data: {
    labels: ['Compressing', 'Decompressing', 'Overall'],
    datasets: [
      {
        label: 'DigitalOcean',
        data: [ 11529, 11258, 11393 ],
        borderColor: Utils.CHART_COLORS.blue,
        backgroundColor: Utils.transparentize(Utils.CHART_COLORS.blue, 0.5),
      },
      {
        label: 'Linode',
        data: [ 9738, 11948, 10843],
        borderColor: Utils.CHART_COLORS.red,
        backgroundColor: Utils.transparentize(Utils.CHART_COLORS.red, 0.5),
      },
      {
        label: 'Vultr',
        data: [ 16615, 14181, 15398 ],
        borderColor: Utils.CHART_COLORS.yellow,
        backgroundColor: Utils.transparentize(Utils.CHART_COLORS.yellow, 0.5),
      },
      {
        label: 'UpCloud',
        data: [ 16503, 14521, 15512 ],
        borderColor: Utils.CHART_COLORS.green,
        backgroundColor: Utils.transparentize(Utils.CHART_COLORS.green, 0.5),
      }
    ]
  },
  options: {
    responsive: true,
    maintainAspectRatio: false,
    scales: {
      y: {
        title: {
          display: true,
          text: 'MIPS'
        }
      }
    },
    plugins: {
      legend: {
        position: 'right',
      },
      title: {
        display: true,
        text: 'CPU - 7z Test'
      }
    }
  }
}
{{< /chart >}}